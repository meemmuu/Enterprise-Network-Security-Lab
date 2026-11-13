# Enterprise-Network-Security-Lab
# Secure HQ and Branch Network Lab

A headquarters site and a branch site connected through three routers, built in Cisco Packet Tracer as the final project of the Networks and Cybersecurity track (2025-2026). The lab covers routing, redundancy, VLAN design and layer 2 security, and every device is configured and documented.

## Contents
| File | Description |
| --- | --- |
| `Project.pkt` | Packet Tracer project |
| `Configuration_Report.pdf` | Running configuration of every device |
| `screenshots/` | Topology and verification screenshots |

## Topology
```
HQ-ASW1-4 -- HQ-DSW1 / HQ-DSW2 -- HQ-CORE -- R1 -- R2 -- R3 -- BR-CORE -- BR-DSW1 / BR-DSW2 -- BR-ASW1-3
                                     |
                              Server network 10.10.10.0/24
```
Each site has three layers: access, distribution (multilayer switches) and core.

## Point-to-point links
| Link | Subnet |
| --- | --- |
| HQ-DSW1 to HQ-CORE | 10.20.20.32/30 |
| HQ-DSW2 to HQ-CORE | 10.20.20.36/30 |
| HQ-CORE to R1 | 105.100.50.0/30 |
| R1 to R2 | 20.20.20.0/30 |
| R2 to R3 | 30.30.30.0/30 |
| R3 to BR-CORE | 205.200.100.0/30 |
| BR-DSW2 to BR-CORE | 10.20.20.40/30 |
| BR-DSW1 to BR-CORE | 10.20.20.44/30 |

## VLANs and addressing
| Site | VLAN | Name | Subnet | Gateway (HSRP) |
| --- | --- | --- | --- | --- |
| HQ | 10 | Engineers | 192.168.10.0/24 | 192.168.10.10 |
| HQ | 20 | Management | 192.168.20.0/24 | 192.168.20.10 |
| HQ | 30 | Finance | 192.168.30.0/24 | 192.168.30.10 |
| HQ | 40 | IT | 192.168.40.0/24 | 192.168.40.10 |
| Branch | 60 | Maintenance | 192.168.60.0/24 | 192.168.60.10 |
| Branch | 70 | HR | 192.168.70.0/24 | 192.168.70.10 |
| Branch | 80 | Sales | 192.168.80.0/24 | 192.168.80.10 |

Distribution switch SVIs use .1 (DSW1) and .2 (DSW2). DHCP excludes .1 to .50 in every VLAN.

## Routing and redundancy
- **OSPF area 0** on all routers, core switches and distribution switches
- **Inter-VLAN routing** with SVIs on the multilayer distribution switches
- **HSRP** per VLAN with a virtual gateway, so clients keep a gateway if one distribution switch fails
- **EtherChannel** between each distribution pair: LACP at HQ, PAgP at the branch
- **Rapid PVST+** on the distribution layer, with the root bridge placed on DSW1 at each site

## Security
- **Port security** on access ports: sticky MAC addresses, `shutdown` violation mode at HQ, `restrict` at the branch
- **DHCP snooping** and **dynamic ARP inspection** on access switches
- **PortFast** on end-device ports only
- **Hardened device access:** encrypted passwords, local user accounts, SSH-only VTY lines, 2-minute exec timeout
- **Named extended ACL** on HQ-CORE protecting the server network

```
ip access-list extended SERVER-ACL
 deny tcp any 10.10.10.0 0.0.0.255 eq www
 deny icmp any 10.10.10.0 0.0.0.255
 permit ip any any
interface GigabitEthernet1/0/4
 ip access-group SERVER-ACL out
```

## Services
- **DHCP:** HQ VLANs use the server at 10.10.10.6 through `ip helper-address`; branch VLANs use pools on the distribution switches
- **Syslog** to 10.10.10.7 and **NTP** from 10.10.10.8 on all devices

## Example: HSRP on an HQ distribution switch
```
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 10.10.10.6
 standby 10 ip 192.168.10.10
 standby 10 priority 120
```

## Verification
Screenshots in `screenshots/`:
- [ ] Full topology
- [ ] `show ip ospf neighbor` and `show ip route ospf`
- [ ] `show standby brief` on both distribution switches
- [ ] `show etherchannel summary`
- [ ] `show port-security interface`
- [ ] `show ip dhcp snooping` and `show ip arp inspection`
- [ ] Ping from HQ to Branch
- [ ] ACL test: ping and HTTP to the servers are blocked, other traffic works

## Lab credentials
Device credentials appear in the configuration files. They are for this Packet Tracer lab only and are not used anywhere real.

## Possible improvements
- Replace type 7 passwords with `enable secret` and `username ... secret`
- Add DAI coverage for every VLAN on all access switches
- Set explicit HSRP priorities on the branch distribution switches

## What I learned
(write 2-3 lines: the hardest part and how you solved it)

## My role
(write 1-2 lines)


## Services
- **DHCP:** HQ VLANs use the server at 10.10.10.6 through `ip helper-address`; branch VLANs use pools on the distribution switches

## What's inside
Everything is packed in `Project-main.zip`. Download it to get:
- `Project.pkt`: the Packet Tracer file
- `configuration.pdf`: device configurations
- `Report.docx`: full report (planning, cabling, budget, schedule)
- **VTP (version 2):** domain `Nass.com`. HQ-DSW1 and BR-DSW1 are the servers and the other switches at each site are clients. Core switches do not take part
- **Wireless:**
  - HQ: a wireless LAN controller (192.168.40.100) manages lightweight APs, with SSIDs and VLAN mapping configured on the controller
  - Branch: standalone APs on access ports, one SSID per VLAN, secured with WPA2
- **Syslog** to 10.10.10.7 and **NTP** from 10.10.10.8 on all devices
- **TFTP backups:** device configurations saved to 10.10.10.9, one unique filename per device
