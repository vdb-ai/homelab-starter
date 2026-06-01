# [Homelab Name] — Network

**Last Updated:** YYYY-MM-DD

---

## 1. Physical Topology

### 1.1 Hierarchy (Top → Down)
```
<!-- Fill in: an ASCII tree from ISP → router → core switch → access switches/APs.
     Annotate any port that is a lockout/outage risk with a ⚠️ and the reason. -->
[ISP]
  └── Router / Firewall (<ip>)
        └── Core Switch (<ip>)
              ├── [P?] <device> (<ip>)
              └── [P?] <device> (<ip>)
```

### 1.2 Logical Architecture
| Property | Value |
| :--- | :--- |
| Topology | <!-- e.g. tree (star-of-stars) --> |
| Segmentation | <!-- e.g. 802.1Q VLAN tag-based isolation --> |
| Backbone Speed | <!-- e.g. 10G SFP+ core, 2.5G access --> |
| DHCP Authority | <!-- which device owns DHCP --> |

---

## 2. Switch Port Assignments

> **Reading guide:** PVID = native/untagged VLAN. Tagged (T) = VLANs carried as 802.1Q trunk. Excluded = VLAN not permitted on port.

### 2.1 <Switch Name> — <Model>
**IP:** `<ip>` | **Model:** <model / port count>

| Port | Connected Device | PVID | Untagged | Tagged |
| :--- | :--- | :--- | :--- | :--- |
| 1 | <!-- device, ⚠️ if risky --> | 1 | 1 | 10, 20, 30, 40 |

<!-- Fill in: one subsection per managed switch. -->

---

## 3. QoS & Traffic Prioritization
<!-- Fill in: your prioritization strategy (e.g. protect voice/video from camera saturation) and any per-port or L3 priority settings. Delete this section if you don't shape traffic. -->

---

## 4. Emergency Recovery
<!-- Fill in: how to regain switch/router management if the firewall is down — which port to use, what static IP to set, what management URL to hit. List the config-safety rules that must never change. -->

---

## 5. <Vendor> UI Configuration Notes
<!-- Fill in: any vendor-specific UI quirks worth remembering when applying port/VLAN changes. Delete if not needed. -->
