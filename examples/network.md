# Homelab — Network

**Last Updated:** 2026-04-28

---

## 1. Physical Topology

### 1.1 Hierarchy (Top → Down)
```
[ISP — 2Gb Fiber]
  └── Firewalla Gold Plus (198.51.100.1) — Router / Firewall / DHCP
        └── Core Switch: Zyxel XMG1915-18EP (198.51.100.79) — Port 15 uplink ⚠️ DO NOT CHANGE PVID 1
              ├── [P2  RJ45] ap-1 GWN7674 — Master (198.51.100.45) [PoE]
              ├── [P4  RJ45] ap-2 GWN7674 — Slave (198.51.100.46) [PoE]
              ├── [P6  RJ45] ap-3 GWN7665 — Slave (198.51.100.47) [PoE]
              ├── [P8  RJ45] ap-4 GWN7665 — Slave (198.51.100.48) [PoE]
              ├── [P9  RJ45] Security Switch GS316EPP (198.51.100.74) — 1G via RJ45 SFP transceiver
              ├── [P10 RJ45] Workstation Switch — Netgear MS305 (Unmanaged, PVID 10, 2.5G)
              ├── [P11 RJ45] node1 eno1 — Mini PC (198.51.100.20/.21) — dual-home enp6s18/enp6s18.40 ⚠️ VLAN 40 tagged
              ├── [P12 RJ45] Management Laptop — direct switch management ⚠️ PVID 1 — DO NOT CHANGE 
              ├── [P13 RJ45] node1 nic-25g — USB 2.5G adapter, PBS VM dedicated (PVID 1 only)
              ├── [P15 RJ45] Firewalla Uplink ⚠️ PVID 1 — DO NOT CHANGE (full trunk)
              ├── [P16 RJ45] Distribution Switch GS308EP (198.51.100.15) — room distribution
              ├── [P17 SFP+] node2 — Minisforum MS-02 (`pve-node2`, 198.51.100.31) — 10G DAC
              └── [P18 SFP+] Reserved — future NAS or 10G expansion

        └── Distribution Switch: Netgear GS308EP (198.51.100.75) — Core Switch Port 16 uplink
              ├── Port 1 → Access Switch 2 GS308Ev4 (198.51.100.73)
              ├── Port 2 → Access Switch 1 GS308Ev4 (198.51.100.72)
              ├── Port 3 → Wall Tablet — Galaxy Tab A9+ (HA Dashboard, VLAN 10)
              ├── Ports 4–7 → Future TBD
              └── Port 8 → Uplink to Core Switch Port 16

        └── Security Switch: Netgear GS316EPP (198.51.100.74)
              ├── Port 1 → NVR (VLAN 30)
              ├── Ports 2–15 → IP Cameras PoE (VLAN 30)
              └── Port 16 → RJ45 SFP transceiver → CAT6 → Core Switch Port 9 (1G uplink)

        └── Access Switch 1: Netgear GS308Ev4 (198.51.100.72)
              ├── Port 1 → Uplink to Distribution Switch
              ├── Port 2 → Outdoor Cameras (VLAN 30)
              ├── Ports 3–4 → IoT Devices (VLAN 40)
              ├── Port 5 → Bluetooth Proxy (bt-proxy-1, VLAN 40)
              ├── Port 6 → IoT Device (VLAN 40)
              ├── Port 7 → Main Device (VLAN 10)
              └── Port 8 → Management (VLAN 1)

        └── Access Switch 2: Netgear GS308Ev4 (198.51.100.73)
              ├── Ports 1–2 → Main Devices (VLAN 10)
              ├── Port 3 → Management (VLAN 1)
              ├── Ports 4–7 → IoT Devices (VLAN 40)
              └── Port 8 → Uplink to Distribution Switch

Stored cold spare: Netgear MS108EUP (198.51.100.71) — removed from active topology 2026-04-28
```

### 1.2 Logical Architecture
| Property | Value |
| :--- | :--- |
| Topology | Tree (star-of-stars) |
| Segmentation | 802.1Q VLAN tag-based isolation |
| Backbone Speed | 10G SFP+ (Core ↔ node2 DAC); 2.5G RJ45 (all other Core Switch ports) |
| PoE — APs | Zyxel XMG1915-18EP Ports 2/4/6/8 (direct PoE++ to all 4 APs) |
| PoE — Cameras | GS316EPP (all camera PoE) |
| PoE — Rooms | GS308EP (room distribution PoE) |
| DHCP Authority | Firewalla (Bridge Mode on all SSIDs) |

---

## 2. Switch Port Assignments

> **Reading guide:** PVID = native/untagged VLAN. Tagged (T) = VLANs carried as 802.1Q trunk.
> Excluded = VLAN not permitted on port.

### 2.1 Core Switch — Zyxel XMG1915-18EP
**IP:** `198.51.100.79` | **Model:** 16-Port 2.5G PoE++ + 2× 10G SFP+
**Ports used:** 12 of 18 (P2/4/6/8/9/10/11/12/13/15/16/17). Reserved: P1/3/5/7/14/18.

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Reserved | 1 | 1 | Excluded |
| 2 | ap-1 GWN7674 — Master (198.51.100.45) [PoE] | 1 | 1 | 10, 20, 30, 40 |
| 3 | Reserved | 1 | 1 | Excluded |
| 4 | ap-2 GWN7674 — Slave (198.51.100.46) [PoE] | 1 | 1 | 10, 20, 30, 40 |
| 5 | Reserved | 1 | 1 | Excluded |
| 6 | ap-3 GWN7665 — Slave (198.51.100.47) [PoE] | 1 | 1 | 10, 20, 30, 40 |
| 7 | Reserved | 1 | 1 | Excluded |
| 8 | ap-4 GWN7665 — Slave (198.51.100.48) [PoE] | 1 | 1 | 10, 20, 30, 40 |
| 9 | Security Switch GS316EPP (trunk — 1G via RJ45 SFP transceiver) | 1 | 1 | 10, 20, 30, 40 |
| 10 | Workstation Switch — Netgear MS305 (unmanaged) | 10 | 10 | Excluded |
| 11 | node1 `eno1` — Proxmox mgmt + HAOS dual-home ⚠️ VLAN 40 tag must persist | 1 | 1 | 40 |
| 12 | Management Laptop — direct switch management ⚠️ PVID 1 always — DO NOT CHANGE | 1 | 1 | Excluded |
| 13 | node1 `nic-25g` — USB 2.5G adapter, PBS VM dedicated | 1 | 1 | Excluded |
| 14 | Reserved | 1 | 1 | Excluded |
| 15 | Firewalla Uplink ⚠️ PVID 1 always — DO NOT CHANGE | 1 | 1 | 10, 20, 30, 40 |
| 16 | Distribution Switch GS308EP (room distribution trunk) | 1 | 1 | 10, 20, 30, 40 |
| 17 (SFP+) | `pve-node2` (198.51.100.31) — 10G DAC | 1 | 1 | 40 |
| 18 (SFP+) | Reserved — future NAS or 10G expansion | 1 | 1 | Excluded |

### 2.2 Distribution Switch — Netgear GS308EP
**IP:** `198.51.100.75` | **Uplink:** Core Switch Port 16

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Access Switch 2 (trunk) | 1 | 1 | 10, 20, 30, 40 |
| 2 | Access Switch 1 (trunk) | 1 | 1 | 10, 20, 30, 40 |
| 3 | Wall Tablet — Galaxy Tab A9+ (HA Dashboard) | 10 | 10 | Excluded |
| 4–7 | Future TBD | 1 | 1 | Excluded |
| 8 | Uplink to Core Switch Port 16 | 1 | 1 | 10, 20, 30, 40 |

### 2.3 Security Switch — Netgear GS316EPP
**IP:** `198.51.100.74` | **Purpose:** Cameras and NVR only — strictly VLAN 30

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | NVR (Reolink RLN16-410) | 30 | 30 | Excluded |
| 2 | Camera — Zone 1 (Duo Floodlight PoE) | 30 | 30 | Excluded |
| 3 | Camera — Entry Doorbell (Reolink Wi-Fi Doorbell — wired Ethernet) | 30 | 30 | Excluded |
| 4 | Camera — Zone 2 (CX810) | 30 | 30 | Excluded |
| 5 | Camera — Zone 3 (Duo 3) | 30 | 30 | Excluded |
| 6 | Camera — Zone 4 (Duo 3 PoE) | 30 | 30 | Excluded |
| 7 | Camera — Zone 5 (NVC-D12M) | 30 | 30 | Excluded |
| 8 | Camera — Zone 6 (NVC-D12M) | 30 | 30 | Excluded |
| 9 | Camera — Zone 7 (NVC-D12M) | 30 | 30 | Excluded |
| 10 | Camera — Indoor Zone 1 (E1 Pro) | 30 | 30 | Excluded |
| 11 | Camera — Indoor Zone 2 (E1 Pro) | 30 | 30 | Excluded |
| 12 | Camera — Zone 8 (NVC-D12M) | 30 | 30 | Excluded |
| 13 | Camera — Zone 9 (NVC-D12M) | 30 | 30 | Excluded |
| 14–15 | Future cameras | 30 | 30 | Excluded |
| 16 | SFP Uplink to Core Switch | 1 | 1 | 10, 20, 30, 40 |

### 2.4 Access Switch 1 — Netgear GS308Ev4
**IP:** `198.51.100.72`

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Uplink to Distribution Switch | 1 | 1 | 10, 20, 30, 40 |
| 2 | Outdoor Cameras | 30 | 30 | Excluded |
| 3 | IoT Device | 40 | 40 | Excluded |
| 4 | IoT Device | 40 | 40 | Excluded |
| 5 | Bluetooth Proxy (bt-proxy-1) | 40 | 40 | Excluded |
| 6 | IoT Device | 40 | 40 | Excluded |
| 7 | Main Device | 10 | 10 | Excluded |
| 8 | Management Port | 1 | 1 | Excluded |

### 2.5 Access Switch 2 — Netgear GS308Ev4
**IP:** `198.51.100.73`

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Main Device | 10 | 10 | Excluded |
| 2 | Main Device | 10 | 10 | Excluded |
| 3 | Management Port | 1 | 1 | Excluded |
| 4 | IoT Device | 40 | 40 | Excluded |
| 5 | IoT Device | 40 | 40 | Excluded |
| 6 | IoT Device | 40 | 40 | Excluded |
| 7 | IoT Device | 40 | 40 | Excluded |
| 8 | Uplink to Distribution Switch | 1 | 1 | 10, 20, 30, 40 |

---

## 3. QoS & Traffic Prioritization

**Strategy:** Voice/Video Priority (L2 + L3). Protects work/calls on VLAN 10 from camera
stream saturation on VLAN 30 across the 2 Gbps [ISP] circuit.

### 3.1 Layer 2 — Core Switch (XMG1915-18EP) Port Priority
| Port | Device | Priority |
| :--- | :--- | :--- |
| 2, 4, 6, 8 | APs (Wi-Fi calls) | High — P7 |
| 9 | Trunk to Security Switch | Normal — P2 |
| 10 | Workstation | High — P7 |
| 11 | node1 eno1 / Home Assistant | Medium — P4 |
| 12 | Management | High — P7 |
| 13 | node2 MS-02 | Medium — P4 |
| 15 | Uplink to Firewalla | High — P7 |
| 16 | Trunk to access switches | Medium — P5 |
| 17 | node2 (SFP+ DAC) | Medium — P4 |

### 3.2 Layer 2 — Security Switch (GS316EPP) Port Priority
| Port | Device | Priority |
| :--- | :--- | :--- |
| 1 | NVR | Normal — P2 |
| 2–15 | PoE Cameras | Normal — P2 |
| 16 | Uplink to Core | High — P7 |

### 3.3 Layer 3 — Firewalla Gold Plus
| Feature | Setting |
| :--- | :--- |
| Smart Queue | Enabled |
| Video Conferencing | High Priority → All Devices |

---

## 4. Emergency Recovery

### 4.1 If Firewalla Is Down
1. Do NOT use Core Switch Port 10 (Workstation switch — requires Firewalla routing for cross-VLAN access)
2. Plug laptop directly into **Core Switch Port 12** (dedicated management port — PVID 1, no VLAN tagging required)
3. Set laptop static IP: `198.51.100.2` / Subnet `255.255.255.0`
4. Access Core Switch management at `198.51.100.79`

**Normal management (Firewalla up):** Plug into **Core Switch Port 12**, set static IP `198.51.100.2` / `255.255.255.0`, access `198.51.100.79`.

### 4.2 Configuration Safety Rules
| Rule | Detail |
| :--- | :--- |
| Core Switch Port 15 PVID | Must always be PVID 1 — Firewalla uplink; tagged VLANs 10/20/30/40 must remain |
| Core Switch Port 11 VLAN 40 | Must remain tagged — node1 HA dual-home; losing VLAN 40 tag silently breaks HAOS IoT reach |
| Management VLAN | Always VLAN 1 — never reassign |
| AP Management | Always untagged PVID 1 on Core Switch Ports 2/4/6/8 |

---

## 5. Netgear UI Configuration Output Format
When generating port-configuration instructions for Netgear switches (GS308Ev4, GS308EP, GS316EPP), **never list config by port number.** Group instructions by VLAN ID — the Netgear "Advanced 802.1Q" UI workflow walks one VLAN at a time, selecting which ports are Tagged (T), Untagged (U), or Excluded for that VLAN.

**Example output shape:**

> **For VLAN 10 (Main):**
> - **Tag (T):** Ports 1, 2, 6, 7, 8
> - **Untag (U):** Port 5
> - **Exclude:** Port 4
>
