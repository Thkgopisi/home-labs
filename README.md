# Zero-Trust Networking & Access Management Home Labs

A hands-on home lab that prototypes a secure, multi-VLAN enterprise network. The lab tests perimeter defenses, role-based access management and network segmentation, and uses packet analysis to verify that unauthorized cross-VLAN traffic is identified and blocked.

**Tech:** Cisco IOS · Wireshark · AAA · ACLs · VLAN segmentation

---

## Overview

Traditional flat networks let a compromised device reach everything else. This lab applies **zero-trust principles** instead: every role sits in its own segment, and traffic between segments is controlled by explicit rules.

The lab simulates a small enterprise with separate zones, including a protected **database zone**. The goal was to confirm that traffic from unauthorized segments could not reach it.

## Objectives

- Design a multi-VLAN enterprise topology with clearly separated zones
- Control access between segments using ACLs and AAA
- Test perimeter defenses and network segmentation
- Use Wireshark to inspect traffic and verify that unauthorized cross-VLAN traffic is identified and blocked
- Protect the simulated database zone from unauthorized access

## Topology

```text
            [Layer 3 Switch / Router]
                       |
        +--------------+--------------+
        |              |              |
    [VLAN 10]      [VLAN 20]      [VLAN 30]
      Users        Management      Database
  192.168.10.0/24 192.168.20.0/24 192.168.30.0/24
```

### VLAN Plan

| VLAN ID | Name | Subnet | Purpose |
|---------|------|--------|---------|
| 10 | Users | 192.168.10.0/24 | End-user devices and standard employee workstations |
| 20 | Management | 192.168.20.0/24 | Administrative access and infrastructure management |
| 30 | Database | 192.168.30.0/24 | Protected database zone |

**Lab environment:** GNS3

---

## Security Design

### 1. Network Segmentation
Each role sits in its own VLAN, with 802.1Q trunking between switches and inter-VLAN routing handled at the Layer 3 switch. Devices in one zone cannot reach another unless a rule allows it.

### 2. Access Control (ACLs)
An extended ACL on the Users VLAN's Switch Virtual Interface (SVI) blocks lateral movement from the Users VLAN to the Database VLAN, while other traffic is still permitted.

```text
! Extended ACL protecting the Database zone
ip access-list extended DENY_LATERAL_MOVEMENT
 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any
!
interface Vlan10
 ip access-group DENY_LATERAL_MOVEMENT in
```

### 3. Role-Based Access Management (AAA)
Authentication, authorization and accounting are configured on the Cisco devices using local AAA and privilege levels. Administrators receive configuration access based on their role instead of sharing one generic login.

### 4. Traffic Inspection
Wireshark captures were taken on the trunk links to inspect packets crossing VLAN boundaries and confirm that the ACL drops the prohibited traffic.

---

## Testing & Results

| # | Test | Expected Result | Actual Result | Status |
|---|------|-----------------|---------------|--------|
| 1 | Users VLAN → Database VLAN | Blocked | Traffic dropped at SVI entry | ✅ Pass |
| 2 | Management VLAN → Database VLAN | Allowed | SSH / ICMP session established | ✅ Pass |
| 3 | Users VLAN → External networks | Allowed | HTTP / DNS traffic permitted | ✅ Pass |

**Key finding:** Wireshark captures confirmed that unauthorized cross-VLAN traffic from the Users VLAN was identified and dropped before it reached the simulated database zone.

<!--
Add screenshots here once they are in the docs/ folder, for example:
![Blocked ping from Users VLAN to Database VLAN](docs/blocked-ping.png)
![Wireshark capture on the trunk link](docs/wireshark-capture.png)
-->

---

## Repository Structure

```text
home-labs/
├── README.md
├── configs/        # Sanitized Cisco IOS configs (switches, router)
├── captures/       # Wireshark .pcapng files
├── docs/           # Topology diagram and screenshots
└── tests/          # Test plan and validation results
```

---

## How to Reproduce

1. Build the topology shown above in GNS3.
2. Create the VLANs, access ports and 802.1Q trunks using the baseline configs in `configs/`.
3. Apply the extended ACL and the local AAA configuration.
4. Run the connectivity tests listed in `tests/` while capturing traffic on the trunk links with Wireshark.
5. Compare your results with the table above.

## Skills Demonstrated

- Cisco IOS configuration (VLANs, trunking, inter-VLAN routing)
- Access Control Lists (ACLs) and local AAA with privilege levels
- Network segmentation using zero-trust principles
- Packet analysis and troubleshooting with Wireshark
- Technical documentation

---

## Author

**Thabang Kgopisi**  
[LinkedIn](https://linkedin.com/in/thabang-k) · [GitHub](https://github.com/Thkgopisi) · thabangkgopisi@gmail.com
