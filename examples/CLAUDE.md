# Homelab — Claude Code Project

## About This Project
These files document the complete home infrastructure including network topology,
VLANs, firewall rules, smart home devices, surveillance, and Home Assistant configuration.
They are the living source of truth for all Claude Code sessions.

**Owner:** [Your Name] | **Location:** [City, State] | **Last Reviewed:** 2026-05-17

---

## Personas & Context
- **[Your Name]:** Tech-savvy, capable of DIY execution.
  Comfortable with networking concepts but newer to Linux/CLI workflows.
- **Family:** Partner [Spouse], children [Child1] and [Child2].
- **WAF Rule:** All solutions must be aesthetically clean, reliable, and usable by [Spouse] and the kids without a learning curve.

---

## Core Principles (Apply to All Advice)
1. **Local-First / Privacy:** Video and automations stay local. No cloud subscriptions.
2. **Reliability over Cleverness:** Wired PoE > Wi-Fi. Proven solutions > bleeding edge.
3. **No Monthly Fees:** Zero-subscription model is the priority.
4. **Value Engineering:** Best-in-class prosumer gear. Avoid cheap junk and enterprise overkill.
5. **WAF:** Clean installs, no visible wires, easy family UX.

---

## Critical Safety Rules
Operational footguns that must be respected unprompted. Topical trivia lives in its owning doc per the File Index — only the items here are pulled into every session's context.

- **Switch Port `<N>` + Port `<M>` are management-lockout risks.** Before suggesting any VLAN change on **Port `<N>`** (node1 HA dual-home — VLAN `<mgmt>` tag must persist) or **Port `<M>`** (firewall uplink — full trunk, PVID 1 must not change), issue a mandatory warning about management lockout risk.
- **Proxmox version is PVE 9.1.9 / Debian 13 Trixie** on both node1 (`198.51.100.21`) and node2 (`198.51.100.31`), confirmed 2026-05-11. Do not assume an older release — training-data cutoff predates PVE 9. Authoritative source: `devices.md` host rows. Verify live with `ssh <host> "pveversion && cat /etc/os-release"`.
- **node2 Thunderbolt AER mask is intentional — do NOT "fix" it.** `<workaround-unit>.service` (systemd oneshot, enabled at boot) sets bit 21 of the AER UE Mask register on PCIe bridge `<pcie-bridge-addr>` to suppress benign ACSViol cascades from the Thunderbolt controller's internal peer-to-peer transactions under `intel_iommu=on`. Source of truth: `host-configs/node2/<workaround-unit>.service`. Discovered + fixed 2026-05-17. Verify a behavior change is actually wanted before removing the unit.
- **node2 `/etc/modprobe.d/vfio.conf` binds the iGPU (`<vendor:device-id>`) to `vfio-pci` for VM 200 passthrough.** Removing or commenting this file without first un-setting `hostpci0` on VM 200 (`qm set 200 --delete hostpci0`) will cause VM 200 to fail to start on next boot. Also: `firmware-misc-nonfree` must be installed on VM 200 for the i915 guest driver to load its GuC firmware. Installed 2026-05-19.

---

## Credential Handling
- **Vault scope (hard boundary).** The 1Password service account `<service-account>` is **read-only on the `Automation` vault** and cannot create vaults. `op vault list` returns `Automation` only — Personal/Shared/etc. are unreachable from this machine. Server-enforced; cannot be widened client-side. Claude cannot modify, delete, or rotate any item.
- **Token storage.** Service-account token lives in macOS Keychain (`security find-generic-password -s <keychain-service-name> -w`). `~/.zshenv` defines an `op()` shell function that fetches the token from keychain on demand and injects `OP_SERVICE_ACCOUNT_TOKEN` only for the single `op` invocation — the variable is never present in the parent shell's env or inherited by child processes. **Never re-export the token globally** in `.zshrc`/`.zshenv`/`.profile`; ambient env vars get inherited by every subprocess (npm postinstall, Homebrew formulae, IDE extensions).
- **Primary lookup:** `op read 'op://Automation/<item>/<field>'`. Run `op vault list` first if you're not sure the item exists (the automation vault is named `Automation`, not `homelab`).
- **Touch ID note.** `op` does NOT prompt for Touch ID — that's by service-account design. Touch ID prompts in this workflow come from SSH key signings via the 1P SSH agent during SCP/SSH operations (batched by ControlMaster windows), not from `op`.
- **Fallback:** read directly from config files on host (`/etc/<service>/...`) when the service is already deployed and the value is needed for a cross-check.
- **Never paste passwords into conversation text** — use them inside SSH commands and don't echo them back as confirmations.

---

## Access Matrix

| Target | SSH alias | User | Key | Touch ID | Privileged ops |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Proxmox node 1 | `node1` | root | 1P agent | ✅ | already root |
| Proxmox node 2 | `node2` | root | 1P agent | ✅ | already root |
| Home Assistant OS | `haos` | root | 1P agent | ✅ | already root |
| Services VM (Docker host) | `services` | admin | 1P agent | ✅ | `sudo` → password from `op://Personal/<host>-<user>/password` (paste via ⌘\) |
| GitHub homelab repo | `git@github.com` | git | 1P agent | ✅ | n/a |
| Other repos via HTTPS | n/a | n/a | `gh` CLI keyring token | n/a | unchanged |

All 1P agent entries resolve to the single `Homelab SSH Key` in 1P Personal. Full SSH config semantics, key rotation, recovery procedures, and sudo workflow are in [`access.md`](access.md).

**1P vault rule (hard boundary):**
- **1P Personal vault — DEFAULT** for SSH keys, sudo passwords, and anything pasted interactively. Never put these in the Automation vault.
- **1P Automation vault — INTENTIONAL EXCEPTIONS ONLY** for secrets Claude must `op read` for automated injection into deploy scripts (HA token, Slack webhooks, Grafana admin, etc.). Each Automation-vault entry is a deliberate grant of service-account read access.

The two `op` paths — CLI service account and SSH agent — do not share authorization. The hard-wall to the Automation vault applies to `op` CLI only; the 1P SSH agent reaches Personal (and any vault the unlocked 1P app can see) but never exposes private key text — it only signs SSH handshake challenges.

**First-shot operational patterns** (read before any SSH or `op` command):

- **SSH:** always use the bare alias (`ssh node1 "cmd"`). Never `ssh root@node1` or `ssh 198.51.100.21` — both bypass `~/.ssh/config` and lose user/key/ControlMaster handling.
- **services sudo:** the admin user has NOPASSWD via `/etc/sudoers.d/<user>-nopasswd`. Use `ssh services 'sudo cmd'` directly — never `sudo -S` with piped passwords, never pre-prompt for one.
- **Touch ID timing:** first SSH to each host in a session triggers a Touch ID prompt via the 1P agent; ControlMaster batches the next 10 min silently. A slow first SSH is usually awaiting Touch ID, not a connectivity issue — wait, don't retry.
- **`op` from the interactive Bash tool:** works directly (e.g. `op read 'op://Automation/ha-token/password'`) because `.zshenv`'s `op()` wrapper is loaded by the shell.
- **`op` from a new script you write:** shell functions don't propagate into bash subprocesses. Export the token from Keychain at the top of the script — copy the pattern from `deploy-compose.sh:11–18`.
- **`op` vault scope:** service account is **read-only on the `Automation` vault only**. `op://Personal/...` will silently fail when called by Claude — Personal-vault items are only reachable via the 1P SSH agent (for SSH keys) or interactive user sessions. Do not write code that depends on `op read 'op://Personal/...'`.
- **No `op signin`:** training-data recipes that use `eval $(op signin)` are the pre-service-account interactive model. With keychain-backed service-account mode, `op signin` is wrong.
- **HAOS key rotation:** HAOS has two `authorized_keys` paths — `/root/.ssh/authorized_keys` (runtime, rewritten on reboot) AND `/data/.ssh/authorized_keys` (persistent). Update both or the change vanishes on next reboot.

---

## Document Update Protocol
- **Surgical edits only** — change only the specific items affected; never rewrite sections.
- **Update immediately on confirmation.** Self-trigger after SSH/bash confirms a result, when a specific value lands (IP, port, VM ID, config path, endpoint), and before advancing to the next plan step. Append a dated `CHANGELOG.md` entry in the same pass listing every file:line touched. Never hold a confirmed value only in conversation context.
- **Ask when uncertain — never guess or infer.** Never remove existing entries unless explicitly told to.
- **No doc updates during brainstorming** — only on confirmed changes (e.g., "cable moved", "config saved", "rule applied").
- **Completed projects:** move entry from `projects.md` → `archive/completed-projects.md` (newest at top); also move the project's spec to `docs/superpowers/specs/completed/` and plan to `docs/superpowers/plans/completed/` so the top-level `specs/` and `plans/` directories stay an active backlog; update the plan-path reference inside the new `archive/completed-projects.md` entry to point at `completed/`; leave `✓ PARTIAL` items in `projects.md` until all work is done.

---

## File Index
| File | Contents |
| :--- | :--- |
| `CLAUDE.md` | This file — project context, personas, protocols |
| `projects.md` | Active / upcoming homelab projects — in-flight backlog |
| `CHANGELOG.md` | **Large** — Running log of all physical and config changes |
| `network.md` | Physical topology, switches, port assignments, QoS |
| `vlans.md` | VLAN IDs, subnets, SSIDs, isolation rules |
| `firewall.md` | Firewall rules, DNS blocks, inter-VLAN access policy |
| `devices.md` | All hardware — network gear, APs, VPN clients |
| `access.md` | SSH/sudo/1P access patterns — agent setup, vault rule, key rotation, recovery, sudo workflow |
| `diagnostics.md` | First-look runbook: where to query when something seems wrong (Loki, Grafana, Uptime Kuma, host fallback) |
| `smarthome.md` | Home Assistant server, peripherals, integrations, cameras |
| `automations.md` | Lightweight index of automations and scripts — YAML lives in `ha-automations/` and `ha-scripts/` |
| `ha-automations/*.yaml` | Individual automation YAML files — edit here, deploy via `deploy-ha-automations.sh` |
| `ha-scripts/*.yaml` | Individual script YAML files — edit here, deploy via `deploy-ha-automations.sh` |
| `deploy-ha-automations.sh` | SCP + reload script; fetches HA token from 1Password CLI at runtime |
| `deploy-ha-dashboards.sh` | SCP + Lovelace reload script for dashboards; fetches HA token from 1Password CLI at runtime |
| `deploy-compose.sh` | Rsync + `docker compose up -d` script for VM 200 stacks; `compose/<service>/.env.op-template` resolved via 1Password CLI at runtime |
| `compose/<service>/**` | Docker Compose service trees — edit here, deploy via `deploy-compose.sh <service>` |
| `ha-notes.md` | HA annotations the JSON can't capture: WoL MACs, snapshot conventions, notify group config, SCP deploy workflow, dashboard deploy workflow |
| `dashboards.md` | Lightweight index of managed Lovelace dashboards — YAML lives in `ha-dashboards/` |
| `ha-dashboards/*.yaml` | Individual dashboard YAML files — edit here, deploy via `deploy-ha-dashboards.sh` |
| `archive/` | Historical context — completed project docs, planning artifacts, dated HA `configuration.yaml` snapshots, pre-YAML-mode dashboard snapshots. Read on demand for context on completed work; re-snapshot `configuration.yaml` to `archive/` on HA changes. |
| `archive/completed-projects.md` | Running log of fully completed `projects.md` entries with dates |

### Large File Handling Rule
Files marked **Large** above must NEVER be read in full unless the task requires
editing the entire document. For lookups, searches, or "find references to X":

1. **Use `grep`/`Glob` first** to locate the relevant section, then `Read` only that line range
2. **For multi-file searches**, dispatch the `Explore` sub-agent with a tight scope rather
   than reading large files into the main context

This rule overrides the default Read behavior. If a task genuinely needs the whole file,
state that explicitly before reading.

---

## Change Workflow
1. A confirmed change is triggered by: the owner describing a physical/config change **OR** Claude confirming a command result during active execution
2. Claude makes surgical edits to all affected doc files — only the specific changed items
3. Claude adds a dated entry to the top of `CHANGELOG.md` summarizing the change and listing which files/lines were touched
4. Claude commits the changes; the owner handles `git push` to keep the remote current across machines

**Additional triggers:**
- **New HA automation/script** → create/edit file in `ha-automations/` or `ha-scripts/`, run `./deploy-ha-automations.sh`, update `automations.md` index
- **New HA device/integration** → ha-mcp reflects changes live; no export needed
- **New service (Docker/LXC/VM with web UI or port)** → add Uptime Kuma monitor (Web Services / DNS / TCP Port / Network Infrastructure group) + update `smarthome.md §12`; if cross-VLAN from the services VM, flag a firewall rule first
- **New web UI/service** → add NPM proxy host at `<name>.example.com` (wildcard cert, Force SSL, WebSocket if needed) + add row to Proxy Host Table in `smarthome.md §6`

---

## Session Checkpoints
At the end of each named project phase (or before a major step in a long session):
1. Confirm all phase docs and the CHANGELOG entry are current
2. Claude commits; the owner pushes after the brief is output
3. Output the resume prompt (template below) in a fenced code block so it can be pasted cleanly into a new session

If context feels long and docs are current, proactively offer the resume prompt rather than continuing to accumulate.

### Resume Prompt Template
The artifact is a paste-in prompt for the *next* session — directive, not retrospective. Lean on CHANGELOG for the recap; the prompt only carries what a new session can't reconstruct from files (where to resume, what not to "fix", what needs go-ahead). Fill in the placeholders and output verbatim:

```
Resume <project-name> at Phase <X+1>.

Plan: docs/superpowers/plans/<...>/phase-<X+1>-<name>.md
Read it, then brief me on the next task before taking any action.

Just shipped Phase <X> (<YYYY-MM-DD>) — see CHANGELOG.md for the recap and gates ratified.

Active workarounds — do NOT "fix" without asking:
- <workaround>: <file path>
(or "none")

Wait for explicit go-ahead before:
- <destructive or downtime action>
(or "none")
```

Omit the "Active workarounds" or "Wait for go-ahead" blocks entirely if both lists would be "none" — don't print empty sections.

---

## Plan & Execution Conventions
Override of default `superpowers:writing-plans` and `superpowers:subagent-driven-development` behavior for homelab projects. User instructions take precedence over skill defaults.

### Plan File Structure
- **Phase files, not monoliths.** New plans save as `docs/superpowers/plans/<YYYY-MM-DD-project-name>/README.md` + `phase-N-<short-name>.md`
- **README** is a 30-line nav: goal, list of phases with one-line summaries, validation gates summary, parking-lot decisions
- **Each phase file** = one session's worth of work — target ~300 lines, 1–3 hours of execution
- **Read only the phase file relevant to the current session.** Never read the README + multiple phase files in one session.
- **Existing monolithic plans** stay as-is; only new plans must follow the phase structure. When resuming a monolithic plan, grep for the current task and read just that range — never re-read the full file.
- **Active vs completed location.** Top-level `docs/superpowers/specs/` and `docs/superpowers/plans/` hold only active/in-flight work. Finished projects move to `specs/completed/` and `plans/completed/` at the same time their `projects.md` entry moves to `archive/completed-projects.md` (see Document Update Protocol).

### Per-Task Execution Mode Tagging
Every task in a phase file is tagged with one of:
- **`[DIRECT]`** — execute in main conversation. Default mode. Use for: SSH commands, config file edits, doc updates, git commits, UI handoffs.
- **`[SUBAGENT]`** — dispatch a subagent. Use ONLY when (a) the task produces a logic-bearing artifact worth spec review (HA automation with Jinja2 templates, scripts with conditional logic), OR (b) the task generates 5+ SSH commands of verification chatter (multi-host journalctl reads, post-reboot guest recovery checks).
- **`[HANDOFF]`** — the owner does it. Use for: physical actions, Touch ID, browser UI, anything outside Claude's tool reach. Provide explicit instructions, wait for confirmation.

### Plan Execution Default
- **Default mode is direct in main conversation.** Only invoke `superpowers:subagent-driven-development` when the current phase contains tasks tagged `[SUBAGENT]` AND one of those tasks is about to run — subagent each tagged task individually; everything else continues direct.
- **One phase per session is the target.** If a phase requires multiple sessions, end at a clean validation gate.

### Task Step Structure (homelab variant)
Replace the TDD task structure from `superpowers:writing-plans` with:
1. **Command** to run (exact, with substitutions noted as `<placeholder>` only when truly variable)
2. **Expected output** (specific values or patterns, not vague descriptions)
3. **Verify** (a separate `upsc`/`systemctl`/`journalctl`/`ha_get_state` call confirming the expected end state)
4. **Doc update** (which file:line gets touched, applied immediately per Document Update Protocol)
5. **Commit** at phase boundary or natural checkpoint, not per task

---

## Domain Knowledge — Where to Look
| Question Type | File to Consult |
| :--- | :--- |
| IP address of a device | `devices.md` |
| Which VLAN a device is on | `vlans.md` |
| Switch port assignment | `network.md` |
| Firewall rule or DNS block | `firewall.md` |
| Camera model or NVR config | `smarthome.md` |
| Home Assistant integrations | `smarthome.md` |
| HA automations or scripts | `automations.md` |
| HA dashboard layout or cards | `dashboards.md` + `ha-dashboards/*.yaml` |
| HA entity IDs for building automations | **ha-mcp** (live query via MCP tools) — fallback to `ha-notes.md` if MCP unavailable |
| "X seems broken" / where to start investigating | `diagnostics.md` |
| Loki / Prometheus / Grafana / Uptime Kuma endpoints | `diagnostics.md` |

---

## Upcoming Projects
See [`projects.md`](projects.md) for the active backlog. Completed projects move to
[`archive/completed-projects.md`](archive/completed-projects.md).
