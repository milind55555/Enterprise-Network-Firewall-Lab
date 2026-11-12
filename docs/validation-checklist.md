# Validation and test plan

Record actual results, screenshots, and date after executing tests. Do not mark a test PASS before running it.

| ID | Test | Command / method | Expected result |
|---|---|---|---|
| T01 | VLAN creation at HQ | `show vlan brief` | VLANs 10, 20, 30, 99, 999 exist |
| T02 | HQ trunk | `show interfaces trunk` | Gi0/1 trunk allows 10,20,30,99,999 |
| T03 | Branch VLAN/trunk | Same commands on BR-SW1 | VLANs and trunk match design |
| T04 | Router subinterfaces | `show ip interface brief` | VLAN subinterfaces are up/up |
| T05 | OSPF adjacency | `show ip ospf neighbor` on both routers | One neighbor in FULL state |
| T06 | OSPF learned routes | `show ip route ospf` | Each site learns the other site's VLAN routes |
| T07 | HQ staff DHCP | Set PC to DHCP | Address in `10.10.10.50-200`, gateway `.1`, DNS `10.10.30.11` |
| T08 | Branch staff DHCP relay | Set PC to DHCP | Address in `10.20.10.50-200` via HQ DHCP server |
| T09 | HQ DNS | `nslookup intranet.enterprise-lab.local 10.10.30.11` | Resolves to `10.10.30.20` |
| T10 | Inter-site staff reachability | Ping approved HQ/branch test hosts | Successful if host firewalls permit ICMP |
| T11 | Guest-to-internal isolation | From guest PC, ping/HTTP to staff/server/mgmt IP | Blocked; ACL hit counter increases |
| T12 | Guest DNS | Resolve internal/public hostname using approved DNS | DNS works to the approved resolver |
| T13 | Guest Internet | Browse external site after pfSense NAT is active | Allowed by egress policy |
| T14 | pfSense route | Check routing table | `10.0.0.0/8` via `10.255.10.2` |
| T15 | pfSense outbound NAT | Check NAT states/logs while staff browses | Source translated to WAN interface address |
| T16 | VPN connection | Connect exported OpenVPN profile | Client receives `10.99.0.x` address |
| T17 | VPN allowed service | HTTPS to `10.10.30.20:443` | Successful |
| T18 | VPN unauthorized service | Try unapproved internal host/port | Blocked/logged |
| T19 | SSH management | SSH to router/switch from approved admin VLAN | SSH succeeds; Telnet unavailable |
| T20 | Port security | Connect more than allowed MACs to a secured edge port | Violation counter/action behaves as configured |

## Useful show commands

```text
show ip interface brief
show ip route
show ip ospf neighbor
show ip route ospf
show access-lists
show vlan brief
show interfaces trunk
show port-security
show port-security interface fastethernet 0/1
```

## Evidence layout suggestion

```text
evidence/
├── 01-topology.png
├── 02-hq-vlans-and-trunk.png
├── 03-branch-vlans-and-trunk.png
├── 04-ospf-neighbor-and-routes.png
├── 05-hq-dhcp-lease.png
├── 06-branch-dhcp-lease.png
├── 07-guest-acl-deny-counter.png
├── 08-pfsense-firewall-rules.png
├── 09-pfsense-outbound-nat.png
└── 10-openvpn-allowed-resource.png
```
