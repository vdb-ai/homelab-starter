# homelab-starter

Copy-ready starting points for running a homelab as an **AI-managed system** — the living-documentation approach behind **[vdb-ai.com/homelab](https://vdb-ai.com/homelab)**.

The premise: if your homelab is documented well enough, an AI coding assistant can operate it with you — read the topology, follow your safety rules, make surgical edits, and keep the docs current. This repo is the documentation layer that makes that work, in two forms: **sanitized real examples** and **blank templates**.

## What's here

```
examples/    ← what a mature, lived-in system looks like
templates/   ← your blank canvas
```

- **`examples/`** — redacted versions of a real, in-production homelab's docs. Structure, conventions, and hard-won operational notes are intact; everything identifying (IPs, MACs, hostnames, domains, secrets, names) is replaced with documentation-safe placeholders. Read these to see how far the approach can go.
- **`templates/`** — blank skeletons of the same files: section headings plus `<!-- Fill in: ... -->` guidance, no data. Start here for your own setup.

Read the `examples/` to see what mature looks like, then start from the matching `templates/` file and fill in your own.

## How to use this

**Two paths:**

1. **"Use this template"** — click the green **Use this template** button at the top of the GitHub page to create your own repo seeded with these files. Best if you want a fresh start.
2. **Fork and adapt** — fork (or just copy the individual files you want) into an existing repo. Best if you already have docs and want to adopt the structure piecemeal.

Nothing here connects to any live system — it's documentation structure, not configuration. Delete the files you don't need, rename things to match your gear, and make it yours.

> **Snapshot as of 2026-05.** This is a curated point-in-time snapshot, not a live mirror.

---

For the full story — why AI-managed documentation, how the pieces fit together, and how it's used day to day — see **[vdb-ai.com/homelab](https://vdb-ai.com/homelab)**.
