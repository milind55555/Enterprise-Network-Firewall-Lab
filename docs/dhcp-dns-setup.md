# Central DHCP and DNS services

## HQ static server configuration

Configure the HQ Server-PT (or Linux server) with:

| Setting | Value |
|---|---|
| IP address | `10.10.30.10` for DHCP server; use separate DNS `10.10.30.11` and web `10.10.30.20` if using multiple server objects |
| Mask | `255.255.255.0` |
| Default gateway | `10.10.30.1` |
| DNS | `10.10.30.11` |

If using one Packet Tracer Server-PT for several services, choose a single static address (for example `10.10.30.10`) and update the helper-address, DNS ACL, firewall DNS rules, and DNS values consistently. The repository defaults assume dedicated addresses `.10` DHCP, `.11` DNS, and `.20` web.

## DHCP pools

On HQ-DHCP `10.10.30.10`, create the following pools. Start addresses should avoid infrastructure/static reservations.

| Pool | Default gateway | DNS server | Start address | Mask |
|---|---|---|---|---|
| HQ-STAFF | `10.10.10.1` | `10.10.30.11` | `10.10.10.50` | `/24` |
| HQ-GUEST | `10.10.20.1` | `10.10.30.11` | `10.10.20.50` | `/24` |
| HQ-MGMT | `10.10.99.1` | `10.10.30.11` | `10.10.99.50` | `/24` |
| BR-STAFF | `10.20.10.1` | `10.10.30.11` | `10.20.10.50` | `/24` |
| BR-GUEST | `10.20.20.1` | `10.10.30.11` | `10.20.20.50` | `/24` |
| BR-MGMT | `10.20.99.1` | `10.10.30.11` | `10.20.99.50` | `/24` |

Use a suitable lease duration, such as 8 hours for guest clients and 1 day for staff clients. Keep server VLAN 30 static.

The routers' `ip helper-address 10.10.30.10` statements relay DHCP broadcasts from the client VLANs to the central server. The HQ server must have a valid default gateway and be reachable from the branch over OSPF.

## DNS records

Create internal A records on HQ-DNS `10.10.30.11`:

| Name | Type | Address |
|---|---|---|
| `intranet.enterprise-lab.local` | A | `10.10.30.20` |
| `dhcp.enterprise-lab.local` | A | `10.10.30.10` |
| `dns.enterprise-lab.local` | A | `10.10.30.11` |

Use `intranet.enterprise-lab.local` for the VPN connectivity test. Configure DNS forwarding/recursion to a trusted resolver if Internet name resolution is required. Guest VLAN ACLs explicitly permit DNS only to `10.10.30.11`.

## Linux equivalent (optional)

For a Linux-based lab, use a DHCP server such as Kea or ISC DHCP where supported, and BIND or dnsmasq for DNS. Ensure firewall rules permit DHCP relay traffic (UDP 67/68) and DNS (UDP/TCP 53). This repo's Cisco helper-address target remains `10.10.30.10` unless you change it.
