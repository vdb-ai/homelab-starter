# Homelab — SSH & Access Reference

Operational reference for SSH, sudo, and 1Password access patterns on this homelab.
See `CLAUDE.md §Access Matrix` for the quick-reference table.

---

## Common Pitfalls (Read First)

Quick reference for first-shot correctness. Deeper rationale in the sections below.

- **SSH alias only.** `ssh node1 "cmd"` is right. `ssh root@node1` or `ssh 198.51.100.21` bypasses `~/.ssh/config` and breaks user/key/ControlMaster handling — also burns extra Touch ID prompts.
- **Services-VM sudo is NOPASSWD.** `ssh services 'sudo cmd'` works without prompting. No `sudo -S`, no piped passwords. See §Sudo Workflow for the rare manual-fallback case.
- **Touch ID first, then batched.** First SSH per host triggers Touch ID via the 1P agent; the next 10 min are silent via ControlMaster. A slow first connection is usually awaiting the prompt — wait, don't retry.
- **`op` from Bash tool (interactive):** works directly via `.zshenv`'s `op()` wrapper function.
- **`op` from a new script:** the wrapper is a zsh shell function and does **not** propagate to bash subprocesses. Copy the keychain-export header from `deploy-compose.sh:8–18` into any new script that calls `op`.
- **`op` vault scope is hard-walled.** The service account sees the `Automation` vault read-only and nothing else. `op://Personal/...` will silently fail — Personal-vault items are only reachable via the 1P SSH agent (for SSH key signing) or via your interactive sessions. Never write code that depends on `op read 'op://Personal/...'`.
- **`op signin` is the old model.** Training-data recipes with `eval $(op signin)` predate the service-account migration. The token comes from macOS Keychain now (`security find-generic-password -s <keychain-service-name> -w`).
- **HAOS has two `authorized_keys` paths.** Runtime is `/root/.ssh/authorized_keys`; persistent is `/data/.ssh/authorized_keys`. Runtime gets rewritten from persistent on reboot — update both during any key rotation.

---

## How the 1Password SSH Agent Works

The 1P desktop app runs a local SSH agent socket at:

```
~/Library/Group Containers/<team-id>.com.1password/t/agent.sock
```

`~/.ssh/config` sets `IdentityAgent` to this socket for all hosts (`Host *`). When an SSH
connection is made, OpenSSH asks the 1P agent to sign the handshake challenge. The 1P agent
prompts for Touch ID (or Apple Watch) to authorize the signing — the private key text never
leaves the 1P agent process.

**`IdentitiesOnly yes`** (set globally) tells OpenSSH to only offer keys the agent presents,
ignoring any `~/.ssh/id_*` files. Combined with `IdentityFile ~/.ssh/homelab.pub`, this tells
the agent which key to offer without enabling fallback to a local private key.

**ControlMaster** (`ControlMaster auto`, `ControlPersist 10m`) means the first SSH connection
to a host opens a master connection; subsequent connections over the next 10 minutes multiplex
through it without re-authenticating. This batches Touch ID prompts — typically one per host
per session, not once per command.

---

## Why a non-root user on the Services VM (not root)

Docker volumes on Linux are owned by the UID of the process that creates them. Running compose
stacks as root (`uid=0`) creates volume directories owned by root. If those paths are later
accessed by a container running as a non-root UID, or mounted into a non-root context, ownership
mismatches cause bind-mount failures or permission errors.

The admin user on the Services VM exists to ensure Docker volumes and compose-managed files are
owned by a stable non-root UID (`uid=1000`). The NOPASSWD sudo grant (`/etc/sudoers.d/<user>-nopasswd`)
lets Claude run privileged commands without an interactive password prompt during automated
sessions. A sudo password is still required for your own interactive sessions — see §Sudo Workflow.

> **Lesson learned:** a past incident where running compose as root with `~/` paths bind-mounted
> `/root/` instead of the expected home directory caused live data loss in a reverse-proxy
> container. Run compose as the stable non-root user, and use absolute bind-mount paths.

---

## Why `root` on Proxmox and HAOS

**Proxmox (node1, node2):** `pveadm`, `qm`, `pct`, `pvesm`, and most Proxmox CLI tooling assume
root. Running as a non-root user requires explicit `sudo` wrapping for every management command
and breaks some commands entirely (e.g., `pve-manager` dbus calls). Proxmox clusters also
exchange host keys as root — non-root SSH users create a parallel key management problem.

**HAOS (haos):** The HAOS SSH add-on only provisions a root account. There is no mechanism to
add a non-root user to the HAOS container without bypassing the add-on. The tradeoff is
accepted — HAOS has no persistent package system and the SSH surface is limited to the LAN.

---

## Key Rotation Procedure

Covers the case where the single `Homelab SSH Key` in 1P Personal needs to be replaced.

1. **Generate new key in 1Password:** 1P → Personal vault → `Homelab SSH Key` → Edit → regenerate
   Ed25519 key. Note the new public key fingerprint.
2. **Push new public key to all 5 endpoints** (while the old key still works):
   - `node1`: `ssh node1 "echo '<new-pubkey>' >> /root/.ssh/authorized_keys"`
   - `node2`: same pattern at `/root/.ssh/authorized_keys`
   - `haos`: same pattern at `/root/.ssh/authorized_keys` AND `/data/.ssh/authorized_keys`
     (HAOS rewrites the runtime path on reboot from persistent storage)
   - `services`: `ssh services "echo '<new-pubkey>' >> /home/<user>/.ssh/authorized_keys"`
   - GitHub: Settings → SSH and GPG keys → New SSH key → paste public key
3. **Verify new key works** against each endpoint before revoking the old one.
4. **Revoke old key from all 5 endpoints:** remove the old pubkey line from each
   `authorized_keys` file; delete the old key from GitHub SSH keys.
5. **Update `~/.config/1Password/ssh/agent.toml`** if the item name changed (current entry
   references `item = "Homelab SSH Key"`).

---

## Recovery Procedures

### 1Password unavailable

SSH agent requires the 1P desktop app to be unlocked. If 1P is down or the workstation is unavailable:

| Host | Out-of-band access |
| :--- | :--- |
| node1 | Physical KVM at the rack or iDRAC/IPMI if available |
| node2 | Physical KVM or Thunderbolt console |
| HAOS | `homeassistant.local:8123` web UI → Settings → Add-ons → Terminal & SSH → reset key |
| services | Proxmox web UI on node2 (`198.51.100.31:8006`) → VM 200 → Console |

### Key suspected compromised

Rotate immediately:
1. Generate a new key in 1P (see Key Rotation Procedure above).
2. Push the new public key to all 5 endpoints.
3. Revoke the old key from all 5 endpoints simultaneously (do not wait).
4. Check `~/.ssh/known_hosts` for any anomalies.

---

## Convention for New Hosts

When adding a new homelab host:
1. Add a `Host <alias>` block to `~/.ssh/config` with `HostName` and `User` only — no
   `IdentityFile`, no per-host `IdentityAgent` (the `Host *` block already handles both).
2. Add the `Homelab SSH Key` public key to the host's `authorized_keys`.
3. Do NOT create a separate 1P SSH key item per host — one key serves all homelab targets.
4. ControlMaster is already on globally; no per-host override needed.

---

## 1P Vault Rule

Two independent paths reach 1Password from this workstation:

| Path | Vaults reachable | Auth | Used for |
| :--- | :--- | :--- | :--- |
| `op` CLI (service account) | **Automation vault only, read-only** | Service-account token from macOS Keychain | Automated secret injection into deploy scripts |
| 1P SSH agent | **All vaults the unlocked 1P app can see** | 1P desktop app session + Touch ID | SSH key signing |

**Personal vault — DEFAULT.** SSH keys, sudo passwords, and anything pasted interactively live
here. New secrets go here unless there is a specific reason for Claude automated access.

**Automation vault — INTENTIONAL EXCEPTIONS ONLY.** Only secrets Claude must `op read` for
automated injection: HA long-lived token, Slack webhooks, Grafana admin password, etc. Each
entry is a deliberate grant. Do not put SSH keys or interactive passwords here.

---

## Sudo Workflow on the Services VM

The admin user has `sudo` rights on the Services VM with a password requirement for interactive
sessions. The password is stored at `op://Personal/<host>-<user>/password`.

**To paste interactively:**
1. Press `⌘\` to open 1Password Quick Access.
2. Search `<host>-<user>`.
3. Touch ID to unlock.
4. Copy the password field.
5. Paste at the `[sudo] password for <user>:` prompt.

Claude's automated sudo invocations use the NOPASSWD sudoers grant and do not require this
flow — it's only needed when you are running commands manually in a terminal.
