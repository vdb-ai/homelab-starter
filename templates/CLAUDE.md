# [Homelab Name] — Claude Code Project

<!-- Fill in: one-paragraph description of what these docs cover and that they are the living source of truth for all Claude Code sessions. -->

**Owner:** [Your Name] | **Location:** [City, State] | **Last Reviewed:** YYYY-MM-DD

---

## Personas & Context
<!-- Fill in: who works on this homelab, their skill level, and any family/household constraints (e.g. a "spouse-acceptance-factor" rule). Keep it short — this is what Claude reads to calibrate tone and risk. -->

---

## Core Principles (Apply to All Advice)
<!-- Fill in: 3–6 numbered principles that should shape every recommendation. Examples: local-first/privacy, reliability over cleverness, no monthly fees, value engineering. These are the "why" behind your design choices. -->

---

## Critical Safety Rules
<!-- Fill in: operational footguns that must be respected unprompted — the things that cause an outage or lockout if changed blindly. One bullet per rule, each stating the risk and the "do NOT do X without checking Y" guard. Keep ONLY genuinely dangerous items here; topical detail belongs in its owning doc. -->

---

## Credential Handling
<!-- Fill in: where secrets live and how they are fetched. Cover the vault/secret-manager scope, how tokens are stored, the primary lookup command, and the hard rule "never paste passwords into conversation text." -->

---

## Access Matrix

| Target | SSH alias | User | Key | Touch ID | Privileged ops |
| :--- | :--- | :--- | :--- | :--- | :--- |
| <!-- Fill in: one row per host --> | `<alias>` | `<user>` | `<key/agent>` | ✅/❌ | `<sudo/root note>` |

<!-- Fill in: a sentence on how keys resolve + a pointer to access.md for full detail and the secrets-vault rule. -->

---

## Document Update Protocol
<!-- Fill in: your rules for keeping docs current — surgical edits only, update immediately on a confirmed change, append a dated CHANGELOG entry, and where completed projects move to. -->

---

## File Index
| File | Contents |
| :--- | :--- |
| `CLAUDE.md` | This file — project context, personas, protocols |
| <!-- Fill in: one row per doc in the repo --> | <!-- short description --> |

### Large File Handling Rule
<!-- Fill in: tell Claude never to read large files in full — grep/Glob to the relevant section first, then Read only that range. -->

---

## Change Workflow
<!-- Fill in: the numbered steps from "a change is confirmed" → "docs edited" → "CHANGELOG entry" → "commit". List any additional triggers (new service, new automation, new proxy host) that require specific follow-up actions. -->

---

## Session Checkpoints
<!-- Fill in: what to do at the end of a work session or project phase, and a paste-in "resume prompt" template for the next session. -->

---

## Plan & Execution Conventions
<!-- Fill in: how you want multi-step work planned and executed (phase files vs monoliths, per-task execution tagging, default execution mode). Optional — delete if you don't use a formal planning workflow. -->

---

## Domain Knowledge — Where to Look
| Question Type | File to Consult |
| :--- | :--- |
| <!-- Fill in: map common question types to the doc that answers them --> | `<file>.md` |

---

## Upcoming Projects
<!-- Fill in: pointer to your active backlog (e.g. projects.md) and where completed projects are archived. -->
