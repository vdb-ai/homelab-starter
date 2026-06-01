# Homelab — Device Registry

**Last Updated:** 2026-05-01

---

## 1. Network Infrastructure

| Device | Model | IP Address | Role |
| :--- | :--- | :--- | :--- |
| Router / Firewall | Firewalla Gold Plus | `198.51.100.1` | Router / Firewall / DHCP / WireGuard VPN |
| Core Switch | Zyxel XMG1915-18EP | `198.51.100.79` | 16-Port 2.5G PoE++ + 2× 10G SFP+ — network backbone |
| Distribution Switch | Netgear GS308EP | `198.51.100.75` | House room distribution trunk (uplink → Core Switch Port 16) |
| Security Switch | Netgear GS316EPP | `198.51.100.74` | 16-Port PoE — cameras and NVR only |
| Access Switch 1 | Netgear GS308Ev4 | `198.51.100.72` | Access layer |
| Access Switch 2 | Netgear GS308Ev4 | `198.51.100.73` | Access layer |
| Workstation Switch | Netgear MS305 | No IP | Core Switch Port 10 — VLAN 10 access only |
| Core Switch (cold spare) | Netgear MS108EUP | `198.51.100.71` | Stored — removed from active topology 2026-04-28 |

---

## 2. Access Points

| Device | Model | IP Address | Role | Notes |
| :--- | :--- | :--- | :--- | :--- |
| ap-1 (Primary) | Grandstream GWN7674 | `198.51.100.45` | Master AP + GWN controller | Firmware 1.0.27.6 |
| ap-2 | Grandstream GWN7674 | `198.51.100.46` | Child AP — Failover Master | Firmware 1.0.27.6; failover master if .45 goes down |
| ap-3 | Grandstream GWN7665 | `198.51.100.47` | Child AP | Firmware 1.0.25.43 |
| ap-4 | Grandstream GWN7665 | `198.51.100.48` | Child AP | Firmware 1.0.25.43 |

---

## 3. Static IP Devices

| Device | IP | VLAN | Notes |
| :--- | :--- | :--- | :--- |
| Firewalla Gold Plus | `198.51.100.1` | 1 | Gateway / DNS for all VLANs |
| Core Switch Zyxel XMG1915-18EP | `198.51.100.79` | 1 | Management |
| Core Switch MS108EUP (cold spare) | `198.51.100.71` | 1 | Stored — inactive as of 2026-04-28 |
| Access Switch 1 GS308Ev4 | `198.51.100.72` | 1 | Management |
| Access Switch 2 GS308Ev4 | `198.51.100.73` | 1 | Management |
| Security Switch GS316EPP | `198.51.100.74` | 1 | Management |
| Distribution Switch GS308EP | `198.51.100.75` | 1 | Management |
| ap-1 GWN7674 (Master) | `198.51.100.45` | 1 | Management |
| ap-2 GWN7674 | `198.51.100.46` | 1 | Management |
| ap-3 GWN7665 | `198.51.100.47` | 1 | Management |
| ap-4 GWN7665 | `198.51.100.48` | 1 | Management |
| Proxmox Host — node1 | `198.51.100.21` | 1 | Proxmox VE 9.1.9 — MAC `aa:bb:cc:dd:ee:01` — HP Elite Mini G9 (Intel Core i5-13500, 14c/20t) — 48GB DDR5-4800 SODIMM (32GB + 16GB modules, board max 64GB) — 2TB boot NVMe (primary M.2) + 2TB secondary NVMe (Expansion Slot 1, PBS datastore — passed to VM 103 as scsi1, serial `<serial>`); read-only API token `monitor@pve!pve-exporter` (role PVEAuditor, no privilege separation) — used by prometheus-pve-exporter on VM 200; node_exporter on :9100 (prometheus-node-exporter pkg v1.9.0, systemd); Promtail 3.6.11 systemd service active — ships /var/log/journal/ to Loki at 198.51.100.30:4444 |
| Proxmox Host — node2 | `198.51.100.31` | 1 | Proxmox VE 9.1.9 — MAC `aa:bb:cc:dd:ee:02` (vmbr0 / SFP+ nic2) — Minisforum MS-02 (Intel Core Ultra 9 285H, integrated GPU) — 64GB DDR5-5600 SODIMM (2× 32GB); 2TB boot NVMe — connected to Core Switch P17 via SFP+ 10G DAC (trunk: PVID 1 + VLAN 40 tagged); read-only API token `monitor@pve!pve-exporter` (role PVEAuditor, no privilege separation) — used by prometheus-pve-exporter on VM 200; node_exporter on :9100 (prometheus-node-exporter pkg v1.9.0, systemd); Promtail 3.6.11 systemd service active — ships /var/log/journal/ to Loki at 198.51.100.30:4444; `<workaround-unit>.service` systemd oneshot enabled (masks ACSViol bit in AER UE Mask on bridge `<pcie-bridge-addr>` — silences benign Thunderbolt peer-to-peer ACS violations, source in `host-configs/node2/`, see CHANGELOG 2026-05-17); **iGPU (`<igpu-pci-addr>`, `<vendor:device-id>`) bound to `vfio-pci`** via `/etc/modprobe.d/vfio.conf` (`options vfio-pci ids=<vendor:device-id>`, `softdep i915/xe pre: vfio-pci`) — passed through to VM 200 as `hostpci0`; host has no `/dev/dri/` (intentional — headless, GPU dedicated to guest, see CHANGELOG 2026-05-19) |
| Home Assistant VM | `198.51.100.20` | 1 | HAOS 17.1 (VM 100 on node1) — management + NVR |
| AdGuard Home — Primary | `198.51.100.22` | 1 | LXC CT 101 on node1 — network-wide DNS filtering; Firewalla WAN primary DNS |
| AdGuard Home — Secondary | `198.51.100.32` | 1 | LXC CT 201 on node2 (`pve-node2`) — secondary DNS for failover; Firewalla WAN secondary DNS; kept in sync with primary via `adguardhome-sync` container on VM 200 (every 15 min) |
| Services VM | `198.51.100.30` | 1 | VM 200 on node2 — Debian 13 (Trixie); **28GB RAM** / 8 cores (bumped 24→28 GiB 2026-05-19 for iGPU passthrough headroom); **`hostpci0: <igpu-pci-addr>,pcie=1`** (iGPU); `/dev/dri/renderD128` present in guest (group `render`); VA-API: Intel iHD driver, `va_openDriver() returns 0`, H264+HEVC+AV1 `VAEntrypointVLD` confirmed; `firmware-misc-nonfree` installed for the iGPU GuC/HuC firmware; Docker host for Frigate, MQTT, Immich, Immich Kiosk, NPM, Uptime Kuma; 256GB boot disk (`scsi0` on node2 `local-lvm` thin pool; resized 64G→256G 2026-05-14, root partition `/dev/sdb2` now 251G ext4, swap on `/swapfile` 2G; Docker log rotation defaults set in `/etc/docker/daemon.json` to `max-size=200m, max-file=5`) + 4TB WD SN7100 NVMe at `/mnt/data` (`scsi1`, `backup=0`); USB AI accelerator unbound 2026-05-23 (OpenVINO cutover — restore: `qm set 200 -usb0 host=<port>` + restart); static IP reserved in Firewalla (MAC `aa:bb:cc:dd:ee:03`) — migrated from node1 VM 102 in P6 (2026-05-08); SSH alias `services` defined in `~/.ssh/config` (1Password agent + ControlMaster 10m); `node_exporter` on `:9100` (systemd, `prometheus-node-exporter` pkg v1.9.0); **all critical Docker containers use `restart: always`** (2026-05-19 audit — prometheus, grafana, cadvisor, pve-exporter, blackbox, loki, promtail, frigate, uptime-kuma, npm, mosquitto, immich-*, adguardhome-sync; `it-tools` only service left at `unless-stopped`) |
| Proxmox Backup Server | `198.51.100.43` | 1 | VM 103 on node1 (`pve-node1`) — PBS 3.x, kernel 7.0.2-2-pve; 32GB boot disk (`scsi0`, `local-lvm:vm-103-disk-1`); 2TB NVMe datastore (`scsi1` → `/dev/sdb1`, ext4 at `/mnt/pbs-datastore`); MAC `aa:bb:cc:dd:ee:04` on `vmbr1` (node1 USB 2.5G adapter `nic-25g` — dedicated PBS backup bridge, not on main `vmbr0`); Firewalla reserved + labeled `PBS (node1 VM 103)`; `node_exporter` on `:9100` (prometheus-node-exporter pkg v1.9.0, systemd) |
| Home Assistant (IoT virtual) | `198.18.1.20` | 40 | HA dual-home — IoT interface (`enp6s18.40`) |
| NVR Reolink RLN16-410 | `198.18.0.123` | 30 | Static — Camera VLAN. Frigate authenticates via scoped `frigate-rtsp` General Account (RTSP read-only, no admin rights) — credential in 1P `Automation` vault at `op://Automation/frigate-rtsp/password`. HA Reolink integration + mobile apps + Reolink desktop client still use the admin account (admin password lives in the Personal 1P vault — out of the service-account's reach; rotation pending in Frigate Phase 0c). |
| Emergency Admin Laptop | `198.51.100.2` | 1 | Manual static — Core Switch Port 12 (dedicated management port, always available) |

---

## 4. WireGuard VPN Clients

| Device | VPN IP |
| :--- | :--- |
| laptop-1 | `198.18.9.3` |
| laptop-2 | `198.18.9.6` |
| phone-1 | `198.18.9.2` |
| tablet-1 | `198.18.9.4` |

---

## 5. Firewalla Device Groups

| Group | Purpose | VLAN |
| :--- | :--- | :--- |
| `Legacy Cameras` | Older cameras being phased out as they are replaced | 30 |
| `Reolink Cameras (no internet)` | All Reolink cameras — outbound internet blocked | 30 |
| `IoT Devices` | General IoT — internet access allowed | 40 |
| `IoT Devices Internet Block` | Smart plugs, feeders, garage — outbound internet blocked, HA-local only | 40 |
| `Primary household devices` | Primary users' personal devices | 10 |

---

## 6. IoT Devices (VLAN 40)

| Device | IP | Group | Notes |
| :--- | :--- | :--- | :--- |
| garage-door-1 (Main Garage Door) | `198.18.1.80` | IoT Devices Internet Block | Garage door controller (ESPHome) |
| garage-door-2 (Side Garage Door) | `198.18.1.81` | IoT Devices Internet Block | Garage door controller (ESPHome) |
| IoT pet feeder 1 | `198.18.1.170` | IoT Devices Internet Block | Tuya — see reconnect note below if unavailable in HA |
| IoT pet feeder 2 | `198.18.1.172` | IoT Devices Internet Block | Tuya — see reconnect note below if unavailable in HA |
| Smart Plug 1 | `198.18.1.160` | IoT Devices Internet Block | Tuya/HA |
| Smart Plug 2 | `198.18.1.161` | IoT Devices Internet Block | Tuya/HA |
| Smart Plug 3 | `198.18.1.162` | IoT Devices Internet Block | Tuya/HA |
| Smart Plug 4 | `198.18.1.163` | IoT Devices Internet Block | Tuya/HA |
| Media Streamer 1 | `198.18.1.164` | IoT Devices | — |
| Media Streamer 2 | `198.18.1.165` | IoT Devices | — |
| Network Printer | `198.18.1.98` | IoT Devices | — |
| Smart Heater | `198.18.1.166` | IoT Devices | Tuya |
| Game Console | `198.18.1.167` | IoT Devices | — |
| Smart TV 1 | `198.18.1.168` | IoT Devices | — |
| Smart TV 2 | `198.18.1.169` | IoT Devices | — |
| Smart TV 3 | `198.18.1.181` | IoT Devices | — |
| Fitness Equipment | `198.18.1.182` | (ungrouped) | — |
| bt-proxy-1 | `198.18.1.183` | IoT Devices Internet Block | ESPHome Bluetooth proxy + environmental sensors; Access Switch 1 Port 6 |

**Tuya Feeder Reconnect Procedure** (if feeder entities show `unavailable` in HA):
1. In Firewalla, pause the `IoT Devices Internet Block` rule temporarily
2. Unplug the feeder, wait ~10 seconds, plug it back in
3. Wait ~60 seconds — the feeder phones home to Tuya cloud on boot to establish its session
4. Entities will return to normal state in HA; re-enable the Firewalla block rule
> The block rule can go back on immediately once the entities are back — the feeder maintains its local HA connection after the initial cloud handshake.

---

## 7. Client Devices (VLAN 10 — Main)

| Device | IP | Group | Notes |
| :--- | :--- | :--- | :--- |
| laptop-1 (wired) | `192.0.2.31` | Primary household devices | Ethernet — work machine |
| laptop-2 (Wi-Fi) | `192.0.2.32` | Primary household devices | Wireless — personal machine, daily driver |
| laptop-3 (Wi-Fi) | `192.0.2.34` | Primary household devices | Wireless — semi-retired, occasional use |
| phone-1 | `192.0.2.237` | Primary household devices | — |
| laptop-4 | `192.0.2.33` | Primary household devices | — |
| phone-2 | `192.0.2.187` | Primary household devices | — |
| tablet-1 | `192.0.2.200` | (ungrouped) | HA Dashboard tablet — hardlined via Distribution Switch port 3 |
| Speaker 1 | `192.0.2.236` | (ungrouped) | Planned: migrate to VLAN 40 — see smarthome.md §7 |
| Speaker 2 | DHCP | (ungrouped) | Planned: migrate to VLAN 40 |
| Speaker 3 (x2) | DHCP | (ungrouped) | Planned: migrate to VLAN 40 |

---

## 8. Guest Network Devices (VLAN 20)

| Device | IP | Notes |
| :--- | :--- | :--- |
| Smart Display | `203.0.113.24` | Planned move to VLAN 40 — on Guest temporarily |

---

## 9. Power Equipment

| Device | Model | Location | Notes |
| :--- | :--- | :--- | :--- |
| Dedicated circuit | 20A circuit + 2× NEMA 5-20R outlets | Top rack area | Single-purpose circuit; **top outlet → PDU**; **bottom outlet → UPS inlet** |
| UPS | CyberPower CP2000PFCRM2U | Rack-mounted | 2000VA/1200W PFC Sinewave; 2U; AVR; USB management port (→ node2 for NUT). **Critical (4/4):** Firewalla, Zyxel XMG1915-18EP, node1, node2. **Non-critical/battery (3/4):** GS316EPP, GS308EP, Reolink NVR. NUT server live on node2 (`homelab-ups@198.51.100.91:1234`); node1 connected as SECONDARY. Firmware bcdDevice: 2.00, serial: `<serial>`. |
| PDU | VEVOR 10-Outlet 1U Rackmount Power Strip | Rack-mounted, top slot | 10 individually switched outlets; 110–125V/15A; plugs into top 20A outlet. Powers: KVM adapter, USB charging station. |
| USB Charging Station | 50W 10-Port (6× USB-A + 4× USB-C) | Rack / PDU | Plugged into PDU; powers all 4× AC Infinity rack fans. |
| Rack Fans | 2× AC Infinity MULTIFAN S7 Quiet Dual 120mm USB | Rack (1 unit top, 1 unit bottom) | 2 fans per unit, 4 fans total; USB-powered via charging station. |

---

## 10. A/V & KVM Equipment

| Device | Model | Location | Notes |
| :--- | :--- | :--- | :--- |
| KVM HDMI Extender | UHD-EXB400-KVM | Remote room | HDBaseT over single CAT5e/6A/7 — up to 400 ft. Transmits 4K@60Hz HDMI + USB keyboard/mouse. **Point-to-point only — cannot route through network switches.** Currently: PC port on NVR (transmitter) → remote TV (receiver) as CCTV station. |
