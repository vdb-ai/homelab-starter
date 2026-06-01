# [Homelab Name] — Active Projects

This file tracks in-flight and upcoming homelab work — the current backlog of changes you're
actively working through. It's the companion to `CHANGELOG.md`: this file is what you *intend*
to do, the changelog is the dated record of what actually shipped.

**Workflow:**
- New work is added to the bottom (or wherever it groups logically)
- Mark partially-done multi-phase work `✓ PARTIAL` and leave it here until all phases are done
- When a project is **fully complete**, move its entry to `archive/completed-projects.md`
  (newest at top) and remove it from this file
- Each entry is a short brief: **bold name** — what it does, what it depends on, and which
  docs to update when it's done. Write enough that a fresh Claude Code session can pick it up.

---

## Active / Upcoming Projects

<!--
The three entries below are seeded starter projects — the natural first things to tackle when
adopting this AI-managed-documentation approach. They also double as worked examples of the
entry format. Replace them with your own backlog as you go.
-->

- **Write your `CLAUDE.md`** — the foundation every Claude Code session reads first. Start from
  `templates/CLAUDE.md` and fill in project context, personas, your core principles, the
  **critical safety rules** (the footguns that cause an outage or lockout if changed blindly),
  the access matrix, and the document-update protocol. Until this exists, the assistant has no
  map and no guardrails. When done: this becomes the project's source of truth — every other doc
  hangs off its File Index.

- **Document your network and devices** — give the assistant the topology it needs before it
  touches anything. Start from `templates/network.md`, `templates/vlans.md`, and
  `templates/devices.md`: capture the physical topology, VLAN/subnet scheme, switch port
  assignments, and a device registry (model, IP, role, and the rich notes — host specs,
  monitoring endpoints, known workarounds). When done: update the File Index in `CLAUDE.md` and
  add a `CHANGELOG.md` entry.

- **Stand up the change workflow + first backup** — make the documentation *living*, then make
  it safe to act. Adopt the `CHANGELOG.md` discipline (start from `templates/CHANGELOG.md`),
  write your access and credential patterns (`templates/access.md`), and get one reliable,
  tested backup running **before** automating anything destructive. When done: note the backup
  target in `devices.md` and record the restore-test result in `CHANGELOG.md`.
