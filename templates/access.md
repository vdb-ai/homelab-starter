# [Homelab Name] — SSH & Access Reference

<!-- Fill in: one line on what this file is. See CLAUDE.md §Access Matrix for the quick-reference table. -->

---

## Common Pitfalls (Read First)
<!-- Fill in: the first-shot-correctness bullets. The things that, if done wrong, waste time or break access — e.g. "use the SSH alias, not user@ip", "sudo is NOPASSWD here", "Touch ID batches via ControlMaster — wait, don't retry". Deeper rationale goes in the sections below. -->

---

## How the SSH Agent / Key Flow Works
<!-- Fill in: how keys are presented and signed (agent socket, hardware token, Touch ID), what your ~/.ssh/config sets globally, and how connection multiplexing (ControlMaster) batches auth prompts. -->

---

## Why <user> on <host class> (not root)
<!-- Fill in: the rationale for each user choice — why a non-root user on some hosts (UID stability for container volumes, etc.) and why root on others (tooling assumptions, appliance constraints). This "why" is the value; capture it. -->

---

## Key Rotation Procedure
<!-- Fill in: step-by-step to replace the key — generate, push to all endpoints while the old key still works, verify, then revoke. List every endpoint that holds the key. -->

---

## Recovery Procedures
<!-- Fill in: out-of-band access per host when the secret manager / agent is unavailable, and the "key suspected compromised" rotate-immediately path. -->

| Host | Out-of-band access |
| :--- | :--- |
| <!-- Fill in: one row per host --> | |

---

## Convention for New Hosts
<!-- Fill in: the checklist for onboarding a new host (config block, push key, no per-host key item, etc.). -->

---

## Secrets-Vault Rule
<!-- Fill in: which secrets live where, and the hard boundary between automated-read secrets and interactive-only secrets. Describe the distinct auth paths (CLI service account vs interactive agent) if they differ. -->

---

## Sudo Workflow
<!-- Fill in: how privileged commands run for automation (NOPASSWD grant) vs interactive sessions (password from the vault), and how to retrieve the password when needed. -->
