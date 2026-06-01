# [Homelab Name] — Home Assistant Notes & Annotations

**Purpose:** Supplemental context that the live entity export can't capture — the institutional knowledge behind your HA setup.

> <!-- Fill in: any standing gotchas a new session must know up front (interface naming after a migration, required network-adapter settings, etc.). -->

## HA Version Info

| Component | Version |
| :--- | :--- |
| Core | <!-- --> |
| Supervisor | <!-- --> |
| Operating System | <!-- --> |
| Frontend | <!-- --> |

---

## Wake-on-LAN MAC Addresses
<!-- Fill in: WoL MACs per device, the broadcast address, and the config key requirement. -->

| Entity | Location | WoL MAC |
| :--- | :--- | :--- |
| <!-- --> | <!-- --> | `<mac>` |

---

## Camera Snapshot Conventions
<!-- Fill in: snapshot save path, local URL pattern, and which entities to use for snapshots. -->

---

## Notification Groups (configuration.yaml)
<!-- Fill in: how your notify groups are defined and which entity IDs to target for push/TTS notifications. -->

---

## Slack / Chat Integration (configuration.yaml)
<!-- Fill in: how alerts post to chat (webhook vs official integration), where the webhook secret lives, and the rotation procedure. Delete if not used. -->

---

## Scene Controller / Remote Automation Patterns
<!-- Fill in: any non-obvious patterns for button/remote-driven automations (e.g. trigger on the timestamp + filter by event_type, button-to-scene mapping, LED indicator entities). This is hard-won knowledge worth recording. -->

---

## Key Entity Notes

| Entity | Note |
| :--- | :--- |
| <!-- Fill in: entities with quirks worth remembering — naming inconsistencies, smart plugs vs media players, "do not rename" flags --> | |

---

## File Share / Samba Add-On
<!-- Fill in: how you reach the HA filesystem (SMB shares, allowed hosts), and any media-folder conventions. Delete if not used. -->

---

## SCP / Deploy Workflow
<!-- Fill in: how automation/script/dashboard changes get pushed to HA and reloaded, including any expected-but-harmless errors (e.g. a read-only token 401 on reload) and the rollback path. -->

---

## configuration.yaml Update Workflow
<!-- Fill in: the pull-edit-diff-push-reload loop for configuration.yaml and how you snapshot it into the repo. -->
