# [Homelab Name] — Diagnostics Quick-Start

> **Purpose:** First-look runbook. When something seems broken, this should get you from question to answer in 1–2 queries — no rediscovery of endpoints, labels, or query syntax.

---

## First move — pick by question type

| Question | Where to look first | How |
| :--- | :--- | :--- |
| "Is X up?" | <!-- e.g. uptime monitor --> | <!-- URL or command --> |
| "What's the error?" | <!-- e.g. log aggregator --> | <!-- query template --> |
| "Resource usage / trend?" | <!-- e.g. metrics dashboards --> | <!-- URL --> |
| "Host-level issue?" | <!-- journalctl / dmesg on the host --> | <!-- ssh alias --> |

<!-- Fill in: a "rule of thumb" sentence on which tool to reach for first. -->

---

## <Primary log/query tool>
<!-- Fill in: the cheapest path to logs. Include how to discover what's there (labels/indexes), a canonical "errors in the last N minutes" query, and common narrowing patterns. Document gotchas (binding addresses, time formats, result limits). -->

---

## Endpoint reference

| Service | URL (external) | URL (internal) | Auth | Notes |
| :--- | :--- | :--- | :--- | :--- |
| <!-- Fill in: one row per observability/admin endpoint --> | | | | |

---

## Host-level fallback
<!-- Fill in: when to SSH to a host directly (stack itself suspect, or issue below the container layer) and the go-to commands once there (dmesg, journalctl, systemctl status). -->

---

## When this file goes stale
<!-- Fill in: what changes should trigger an update to this file (new label, new exporter, endpoint change). Record the last-verified date. -->
