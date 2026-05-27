# Multi-VLAN LAN with Inter-VLAN Routing and ACL Segmentation

A Cisco Packet Tracer lab simulating a small enterprise network with three VLANs (Users, Servers, IoT/Guest), router-on-a-stick inter-VLAN routing, per-VLAN DHCP, and ACL-based segmentation to prevent the IoT/Guest segment from reaching the Users and Servers segments.

Built as part of my hands-on networking practice while pursuing my Master's in Computer Engineering for IoT Systems at Hochschule Nordhausen.

## Goal

Design and configure a segmented Layer-2 / Layer-3 network where:

- Users (corporate workstations) can reach Servers.
- IoT / Guest devices are isolated from both Users and Servers — a common requirement for IoT and BYOD environments.
- Each VLAN receives its IP address automatically via DHCP from the router.

## Topology

```
                    [R1]   (Router 2911)
                     |
                (Trunk: VLAN 10, 20, 30)
                     |
                    [SW1]  (Switch 2960)
                  /  |   |    \
              VLAN 10 VLAN 10 VLAN 20  VLAN 30
                PC1   PC2   Server1   IoT-Pc
```

| VLAN | Name      | Network          | Gateway        | DHCP Range            |
|------|-----------|------------------|----------------|------------------------|
| 10   | USERS     | 192.168.10.0/24  | 192.168.10.1   | 192.168.10.10 – .100  |
| 20   | SERVERS   | 192.168.20.0/24  | 192.168.20.1   | 192.168.20.10 – .100  |
| 30   | IOT_GUEST | 192.168.30.0/24  | 192.168.30.1   | 192.168.30.10 – .100  |

## Devices

- 1 × Cisco 2911 router (R1) — inter-VLAN routing, DHCP server, ACL enforcement
- 1 × Cisco 2960 switch (SW1) — Layer-2 VLAN segmentation, 802.1Q trunking
- 2 × PC — Users VLAN
- 1 × Server — Servers VLAN
- 1 × PC — IoT / Guest VLAN

## Configuration highlights

### Switch (SW1)
- Three VLANs defined: USERS (10), SERVERS (20), IOT_GUEST (30)
- Access ports assigned per device
- Gig0/1 configured as an 802.1Q trunk to R1 with `allowed vlan 10,20,30`

### Router (R1)
- Three sub-interfaces on Gig0/0 (`.10`, `.20`, `.30`) with `encapsulation dot1Q`
- DHCP pools per VLAN with reserved ranges for static infrastructure (.1–.9)
- Extended ACL `IOT_RESTRICT` applied inbound on Gig0/0.30 to drop traffic from the IoT/Guest VLAN destined for the Users and Servers VLANs

Full running configurations are in [`R1-running-config.txt`](R1-running-config.txt) and [`SW1-running-config.txt`](SW1-running-config.txt).

## Testing & verification

| Test                                              | Expected   | Result      |
|---------------------------------------------------|------------|-------------|
| PC1 / PC2 receive DHCP address in 192.168.10.x    | ✅          | ✅           |
| Server1 receives DHCP address in 192.168.20.x     | ✅          | ✅           |
| IoT-Pc receives DHCP address in 192.168.30.x      | ✅          | ✅           |
| PC1 → Server1 ping (Users to Servers)             | Success    | Success ✅   |
| IoT-Pc → Server1 ping (before ACL)                | Success    | Success ✅   |
| IoT-Pc → Server1 ping (after ACL applied)         | Blocked    | Blocked ❌   |
| IoT-Pc → PC1 ping (after ACL applied)             | Blocked    | Blocked ❌   |
| PC1 → Server1 ping (after ACL applied)            | Success    | Success ✅   |

The pre-ACL and post-ACL comparison confirms the ACL is the only thing blocking IoT traffic to the protected segments.

## What I learned

- Practical VLAN design and 802.1Q trunking on Cisco IOS.
- Router-on-a-stick as a Layer-3 gateway for multiple VLANs via sub-interfaces.
- Per-VLAN DHCP pool configuration with reserved ranges for static infrastructure.
- Using extended ACLs with wildcard masks to enforce network segmentation policy — a core technique for isolating IoT and guest traffic from production resources.
- The importance of testing both *positive* (what should work) and *negative* (what should be blocked) cases when validating a security policy.

## How to open

1. Install [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer) (free with a Cisco Networking Academy account).
2. Open `multi-vlan-lan.pkt` in Packet Tracer.

## Skills demonstrated

`Cisco IOS` · `VLANs` · `802.1Q Trunking` · `Inter-VLAN Routing` · `Router-on-a-Stick` · `DHCP` · `Access Control Lists (ACLs)` · `Network Segmentation` · `Network Design`

---

**Author:** Wasif Gull · Master's student in Computer Engineering for IoT Systems @ HS Nordhausen
[LinkedIn](https://www.linkedin.com/in/wasif-gull-257124403/)
