# Homelab — Firewall & Security Rules

**Last Updated:** 2026-04-03

---

## 1. Global DNS Blocks

| Domain | Action | Reason |
| :--- | :--- | :--- |
| `gmial.com` | Block | Typosquat of a popular email domain — part of the default threat bundle, not manually added; likely also covered by AdGuard blocklists |
| `telemetry.example-analytics.com` | Allow | Overrides Default Bundle — intentionally permitted |
| `metrics.example-vendor.com` | Allow | Overrides Default Bundle — intentionally permitted |

---

## 2. Inter-VLAN Access Policy

| Rule | Detail |
| :--- | :--- |
| VLAN 1 (Management) | Strictly infrastructure — no client devices, no inbound from other VLANs |
| VLAN 10 → Management | Primary household devices have inter-VLAN management access via Firewalla rule. |
| VLAN 20 (Guest) | Fully isolated — internet only, no access to any other VLAN |
| VLAN 30 (Cameras) | Fully isolated — NVR and cameras only. No outbound internet except NVR updates when needed. |
| VLAN 40 (IoT) | Block Traffic from All Local Networks — devices cannot initiate connections. HA can reach in via dual-home interface. |

---

## 3. Critical Port Rules

| Rule | Detail |
| :--- | :--- |
| Core Port 8 PVID | Must always be PVID 1 — Firewalla uplink. Changing this causes full network outage. |
| Core Port 4 PVID | Must always be PVID 1 — AP trunk port; usable for emergency admin (unplug AP first) |
| Management VLAN | Always VLAN 1 — never reassign to another VLAN ID |
| AP Management | Always untagged PVID 1 on Core Ports 1–4 |

---

## 4. Layer 3 — Firewalla Rules

> QoS config (Smart Queue, Video Conferencing priority) is documented in `network.md` Section 3.

| Action | Rule / Target | Applied To | Notes |
| :--- | :--- | :--- | :--- |
| Block | Active Protect | All Devices | Firewalla built-in security ruleset (malware, phishing, botnet) — ON, unrelated to AdGuard |
| Block | Ad Block | All Devices | **Disabled 2026-03-25** — AdGuard Home (198.51.100.22) now handles all DNS filtering |
| Block | Traffic from Internet | All Devices | Blocks all unsolicited inbound traffic |
| Block | Traffic to All Local Networks | IoT | IoT VLAN cannot initiate cross-VLAN connections |
| Block | Traffic to All Local Networks | Cameras | Camera VLAN cannot initiate cross-VLAN connections |
| Allow | `telemetry.example-analytics.com` | All Devices | Overrides Default Bundle block |
| Allow | `metrics.example-vendor.com` | All Devices | Overrides Default Bundle block |
| Block | Traffic from & to All Local Networks | Guest | Guest VLAN fully isolated from all local networks |
| Block | `<cloud-server-ip>` | IoT Camera | Manual block — vendor cloud server IP |
| Block | `gmial.com` | All Devices | Firewalla default bundle — see §1 |
| Allow | Traffic from & to Default LAN (VLAN 1 / Management) | Primary household devices | Inter-VLAN management access for admin devices |
| Allow | `198.18.1.98` (Network Printer) | WireGuard | Remote print access via VPN |
| Block | Traffic from & to Internet | Reolink Cameras (no internet) | No cloud — local/NVR only |
| Allow | Traffic from & to Cameras | Reolink Cameras (no internet) | Intra-VLAN 30 communication allowed |
| Allow | `198.18.0.123` (Reolink NVR) | Reolink Cameras (no internet) | Camera → NVR direct access |
| Allow | Traffic from & to Cameras | Primary household devices | Admin access to Camera VLAN |
| Allow | `198.51.100.45` (Grandstream AP Master) | Primary household devices | Outbound — AP management access |
| Block | Traffic from & to Internet | Reolink NVR | NVR has no cloud access |
| Allow | Traffic from & to Cameras | Reolink NVR | NVR intra-VLAN 30 communication |
| Allow | `pushx.reolink.com` | Reolink NVR | NVR push notifications only |
| Allow | `pushx.reolink.com` | Reolink Cameras (no internet) | Camera push notifications |
| Allow | `198.18.1.98` (Network Printer) | Guest | Guest network print access |
| Allow | `198.18.1.98` (Network Printer) | Main | Main VLAN print access |
| Allow | `198.18.0.123` (Reolink NVR) | Primary household devices | Admin direct NVR access |
| Block | Traffic from & to Internet | IoT Devices Internet Block | Outbound internet blocked for this group |

---

## 5. WireGuard VPN Clients
See `devices.md` Section 4 for the full client list.
