# Homelab — Diagnostics Quick-Start

> **Purpose:** First-look runbook. When something seems broken, this gets from
> question to answer in 1–2 queries — no rediscovery of endpoints, labels, or
> query syntax needed.
>
> **Out of scope:** What's monitored (→ `smarthome.md §12`), stack design
> (→ `compose/observability/`, `CHANGELOG.md`), deep service configs
> (→ per-service docs).

---

## First move — pick by question type

| Question | Where to look first | How |
| :--- | :--- | :--- |
| "Is X up?" | **Uptime Kuma** | https://uptimekuma.example.com — visual; or query the API |
| "What's the error?" | **Loki LogQL** | `curl http://198.51.100.30:4444/loki/api/v1/query_range ...` (template below) |
| "When did X start failing?" | **Loki, narrowed** | Add `{container_name=...}` or `{host=...,unit=...}` |
| "How loud is X (events/sec)?" | **Loki**, count by minute | `awk -F'[T:]' '{print $2":"$3}' \| sort \| uniq -c` over query output |
| "Resource usage / hot spot?" | **Grafana** dashboards | https://grafana.example.com |
| "PVE host-level issue?" (kernel, PCIe, hw) | `journalctl` / `dmesg` on the host, or Loki `{host=...,job="journal"}` | `ssh node1` or `ssh node2` |
| "DNS issue?" | **AdGuard query log** | http://198.51.100.22/ (primary) |
| "Network reachability?" | **Blackbox** probes via Grafana | https://grafana.example.com |

**Rule of thumb:** start with Uptime Kuma if it's a service-up question, Loki if it's a log/error question, Grafana if it's a metrics/trend question. Only `ssh` to a host when the observability stack itself is in doubt OR the issue is below Docker (kernel, PCIe, hardware).

---

## Loki — the workhorse

Loki HTTP API is the cheapest path. Hit it **directly from the workstation** — no SSH needed, no auth, no Grafana.

### Discover what's there

```bash
# What labels exist?
curl -s http://198.51.100.30:4444/loki/api/v1/labels | python3 -m json.tool

# What values for a given label?
curl -s http://198.51.100.30:4444/loki/api/v1/label/host/values | python3 -m json.tool
curl -s http://198.51.100.30:4444/loki/api/v1/label/job/values | python3 -m json.tool
```

### Label vocabulary (as of 2026-05-17)

| Label | Values | Notes |
| :--- | :--- | :--- |
| `host` | `node1`, `node2`, `services` | Where the log originated |
| `job` | `docker`, `journal` | `docker` = Services VM containers; `journal` = systemd journal from all 3 hosts |
| `compose_service` | (Docker service name) | Only on `job=docker` |
| `container_name` | (Docker container name) | Only on `job=docker` |
| `unit` | (systemd unit name) | Only on `job=journal` |
| `level` | log level | Extracted from structured logs where possible |

### The canonical "errors in last 10 minutes, everywhere" query

```bash
TEN_MIN_AGO=$(( $(date +%s) - 600 ))000000000
NOW_NS=$(date +%s)000000000

curl -s "http://198.51.100.30:4444/loki/api/v1/query_range" \
  --data-urlencode 'query={job=~".+"} |~ "(?i)(error|warn|fatal|panic|crash|refused|timeout|failed)"' \
  --data-urlencode "start=${TEN_MIN_AGO}" \
  --data-urlencode "end=${NOW_NS}" \
  --data-urlencode "limit=300" \
  --data-urlencode "direction=backward"
```

### Narrowing patterns

| Want | LogQL selector |
| :--- | :--- |
| Just one host's logs | `{host="node2"}` |
| Just one container | `{container_name="frigate"}` |
| Just one compose service | `{compose_service="loki"}` |
| Just one systemd unit | `{job="journal", unit="promtail.service"}` |
| Host kernel/journal only | `{host="node2", job="journal"}` |
| Everything except Docker | `{job="journal"}` |

Add `|~ "regex"` to filter contents, or `|= "literal"` for substring match.

### Gotchas

- **Loki binds to `198.51.100.30:4444`, NOT `127.0.0.1:4444`.** `curl localhost:4444` from inside the Services VM will fail with `exit 7 (Failed to connect)`. Use the external IP even from on-host.
- **Time params take nanoseconds since epoch**, not RFC3339. Multiply Unix seconds by 10^9 (`$(date +%s)000000000`), or queries return empty/weird windows.
- **`limit` is per-stream and the default cap is 300.** For high-volume bursts (e.g. a kernel error storm at hundreds/sec), 300 lines covers maybe 1 second. Shrink the time window or count via aggregation instead of streaming raw lines.
- **Promtail backfills on first start.** If you just deployed a Promtail and see old events flooding in, that's the journal positions file establishing — give it a few minutes to catch up to real-time.

---

## Endpoint reference

| Service | URL (external) | URL (internal) | Auth | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Uptime Kuma | https://uptimekuma.example.com | http://198.51.100.30:4040 | UI login | First "is it up?" check |
| Grafana | https://grafana.example.com | http://127.0.0.1:4443 (Services VM only, behind NPM) | UI login | Dashboards over Loki + Prometheus |
| Loki | (internal only) | http://198.51.100.30:4444 | none | HTTP API; query directly from workstation |
| Prometheus | (internal only) | http://127.0.0.1:5555 (Services VM only) | none | Loopback-bound; use Grafana or SSH tunnel |
| node_exporter | — | `:6666` on node1, node2, services | none | Host CPU/RAM/disk/net |
| pve-exporter | — | `:7777` on services | uses `monitor@pve!pve-exporter` token | Scrapes both Proxmox hosts |
| cAdvisor | — | `:8888` inside compose net | none | Per-container metrics; not exposed on host |
| blackbox | — | `:9999` on services | none | HTTP/TCP/ICMP probe target for Prometheus |
| AdGuard primary | http://198.51.100.22/ | same | UI login | DNS query log; on node1 LXC CT 101 |
| AdGuard secondary | http://198.51.100.32/ | same | UI login | Synced from primary every 15 min; failover only |

---

## Host-level fallback

When the observability stack itself is suspect, OR the issue is below the Docker layer (kernel panics, PCIe errors, hardware), go to the host directly:

```bash
ssh node1           # 198.51.100.21
ssh node2           # 198.51.100.31
ssh services        # admin user via SSH alias (1Password agent + ControlMaster 10m)

# Kernel ring buffer (current boot only, in-memory, finite)
dmesg --time-format=iso | tail -100
dmesg --time-format=iso | grep -i 'error\|warn\|fatal'

# Persistent journal (also shipped to Loki under job=journal)
journalctl --since '10 min ago' -p err --no-pager
journalctl -u <unit-name> -n 100 --no-pager
journalctl -k --since '1 hour ago'   # kernel only

# Service status quick check
systemctl status <unit-name> --no-pager
```

The persistent journal is also shipping to Loki (`{host=..., job="journal"}`), so for content within the journal these are interchangeable — Loki is faster across hosts, `journalctl` is faster for tight time windows on a single host.

The dmesg in-memory ring buffer is bounded (~1 MB by default); for older kernel events, query Loki's `journal` job — systemd-journald captures kernel messages too.

---

## When this file goes stale

This file documents endpoints, labels, and query patterns that change when the observability stack changes. Update it when:
- A new Loki label is added or label cardinality changes meaningfully
- A new exporter / metric source is added
- An endpoint URL or binding changes
- A "first move" answer changes (e.g. Grafana becomes the primary surface over direct curl)

Last verified against live stack: **2026-05-17** (Phase 2 of Loki + Promtail project complete; node1 + node2 journals streaming).
