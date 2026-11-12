# Architecture and traffic flows

## Design choice

This reference lab uses router-on-a-stick at both offices. Each access switch carries VLANs 10, 20, 30, and 99 to its site's router. The routers provide default gateways, inter-VLAN routing, DHCP relay, guest isolation ACLs, and OSPF adjacency over a point-to-point routed link.

HQ is the Internet egress site. HQ-R1 has a default route to pfSense. HQ-R1 originates that default route into OSPF so the branch router forwards Internet-bound traffic toward HQ. pfSense performs stateful filtering and source NAT for all routed office networks.

## Main traffic flows

### Staff at HQ to an HQ server

`HQ client (10.10.10.x) -> HQ-SW1 -> HQ-R1 VLAN 10 gateway -> HQ-R1 VLAN 30 gateway -> HQ server (10.10.30.x)`

### Staff at Branch to an HQ server

`Branch client (10.20.10.x) -> BR-SW1 -> BR-R1 -> OSPF WAN -> HQ-R1 -> HQ server`

### Branch client to the Internet

`Branch client -> BR-R1 -> OSPF-learned default route -> HQ-R1 -> pfSense LAN -> pfSense WAN (outbound NAT) -> ISP`

### Guest attempting to reach an internal network

The extended ACL is applied inbound on the guest VLAN subinterface. It permits DHCP and DNS to the designated internal DNS server, denies RFC1918 destination ranges, and permits other destinations. Therefore guest Internet traffic can proceed through HQ while internal private destinations are denied.

### Remote VPN client

`Remote client -> Internet -> pfSense WAN UDP/1194 -> OpenVPN tunnel (10.99.0.0/24) -> pfSense static route -> HQ-R1 -> approved HQ subnet/host`

The example VPN policy is split tunnel and grants access to selected HQ resources only. Branch network access is intentionally not included by default.

## Routing summary

- OSPF process ID: `10` (locally significant; both routers use the same value for clarity).
- OSPF area: `0`.
- OSPF WAN network: `10.255.0.0/30`.
- HQ advertises its VLAN networks and a default route learned from pfSense.
- Branch advertises its VLAN networks and learns HQ/Internet routes from HQ.
- pfSense has a static route for `10.0.0.0/8` via HQ-R1 `10.255.10.2`.
- HQ-R1 has a static route for VPN pool `10.99.0.0/24` via pfSense `10.255.10.1`.

## Integration boundary

Packet Tracer is used for the Cisco IOS switching/routing portion. pfSense is a separate VM with its own virtual NICs and upstream network. The transit IP plan is the hand-off contract between those environments. For a fully connected emulation, recreate the same IP plan in GNS3/EVE-NG and connect the pfSense VM to the virtual links.
