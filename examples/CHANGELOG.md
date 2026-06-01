<!-- Format demo — see README for the real system -->
# Homelab — Change Log

> **Format:** Date | Change description | Files affected
> Add new entries at the TOP of this file.

---

## 2026-04-12 — UPS monitoring stack completed

NUT monitoring project finished. Server runs on node2; node1 attached as a secondary. HA
surfaces battery state via three automations (on-battery, shutdown-imminent, power-restored)
with Slack + mobile alerts. Restore test passed — pulled mains, confirmed graceful shutdown
sequence fired at the FSD threshold.

**Files touched:**
- `devices.md §9` — added UPS row (CyberPower, USB management → node2)
- `automations.md` — indexed the three UPS automations
- `diagnostics.md` — added "is the UPS on battery?" first-look entry
- `CHANGELOG.md`

---

## 2026-03-28 — Moved DNS filtering off the router to AdGuard

Disabled the router's built-in ad-block rule; AdGuard Home (LXC on node1) now handles all
DNS filtering as the WAN primary DNS. A secondary AdGuard on node2 provides failover and is
kept in sync every 15 minutes.

**Files touched:**
- `firewall.md §1` — noted router Ad Block disabled, AdGuard now authoritative
- `firewall.md §4` — updated the Layer-3 rule table
- `devices.md §3` — added both AdGuard static-IP rows
- `CHANGELOG.md`

---

## 2026-03-15 — Added wall-mounted dashboard tablet

Mounted a tablet in a common area as a kiosk for the Command Center dashboard. Hardlined via
the Distribution Switch on VLAN 10. Configured Wake-on-LAN and a kiosk-mode auto-launch.

**Files touched:**
- `devices.md §7` — added tablet row (VLAN 10, IP 192.0.2.200)
- `network.md §2.2` — added the Distribution Switch port assignment
- `dashboards.md` — noted the kiosk dashboard target
- `CHANGELOG.md`
