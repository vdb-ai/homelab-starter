# Homelab — VLANs & SSIDs

**Last Updated:** 2026-03-17

---

## 1. VLAN Summary

| VLAN ID | Name | Subnet | SSID | Band | Security | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Management | `198.51.100.x` | None | N/A | N/A | Switches, APs, Router — infrastructure only. No client devices. |
| 10 | Main | `192.0.2.x` | `HomeNet` | Tri-Band | WPA3-SAE | Personal devices, work laptops, gaming |
| 20 | Guest | `203.0.113.x` | `HomeNet-Guest` | Tri-Band | WPA3-SAE | Isolated visitor access |
| 30 | Cameras | `198.18.0.x` | `HomeNet-Cameras` | 2.4/5GHz | WPA2/WPA3 | Security cameras + NVR only (Hidden SSID) |
| 40 | IoT | `198.18.1.x` | `HomeNet-IoT` | 2.4/5GHz | WPA/WPA2 | Smart home devices — no local network access |
| VPN | WireGuard | `198.18.9.x` | None | N/A | WireGuard | Remote access tunnel |

---

## 2. Isolation Rules

| VLAN | Rule | Detail |
| :--- | :--- | :--- |
| VLAN 1 | Management locked | Infrastructure only — no client devices ever (HA VM at `198.51.100.20` and Services VM at `198.51.100.30` are intentional exceptions — treated as infrastructure) |
| VLAN 10 | Primary household devices | Has inter-VLAN management access via Firewalla rule |
| VLAN 20 | Full isolation | No access to any other VLAN |
| VLAN 30 | Full isolation | NVR and cameras only — no outbound except NVR |
| VLAN 40 | No local access | Devices blocked from all local networks. HA reaches in via dual-home |

---

## 3. Access Point SSID Configuration

**Controller:** Grandstream GWN7674 Master (`198.51.100.45`)
**DHCP Mode:** Bridge — Firewalla controls all DHCP
**802.1Q Tagging:** Explicitly enabled in GWN UI

| SSID | VLAN Tag | Band | Security Mode | Notes |
| :--- | :--- | :--- | :--- | :--- |
| `HomeNet` | 10 | Tri-Band | WPA3-SAE | Primary personal network |
| `HomeNet-Guest` | 20 | Tri-Band | WPA3-SAE | Isolated guests |
| `HomeNet-Cameras` | 30 | 2.4/5GHz | WPA2/WPA3 | Hidden SSID |
| `HomeNet-IoT` | 40 | 2.4/5GHz | WPA/WPA2 | Smart home devices |

---

## 4. Home Assistant Dual-Home Configuration

**Strategy:** Router-on-a-Stick topology applied to the HA server. HA exists on multiple
VLANs simultaneously via virtual interfaces, capturing local broadcast traffic (mDNS/SSDP)
natively — eliminating need for cross-VLAN mDNS reflectors.

| Interface | VLAN | IP Address | Gateway/DNS | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `enp6s18` (Primary) | 1 (Untagged/PVID on Core Switch Port 11) | `198.51.100.20` (Static) | `198.51.100.1` (Active) | NVR comms, dashboard access, system management, internet |
| `enp6s18.40` (Virtual) | 40 (Tagged on Core Switch Port 11) | `198.18.1.20` (Static, CLI configured) | Blank (intentional) | Direct local comms with IoT devices |

**HA Network Adapter setting (Settings → System → Network → Network Adapter):** Both `enp6s18` and `enp6s18.40` enabled (confirmed 2026-05-06). HA participates in mDNS/SSDP discovery on both networks and announces at both `198.51.100.20` and `198.18.1.20`. New IoT devices on VLAN 40 will be auto-discovered.

**Strategic benefits:**
- Zero-Trust IoT: Strict "Block Traffic from All Local Networks" on VLAN 40. Devices are trapped, HA reaches in.
- HomeKit / smart thermostats: Instant local IP discovery without cloud polling
