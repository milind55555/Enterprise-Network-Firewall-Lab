# IP addressing plan

All user/server VLANs are `/24`. `.1` is the router gateway. Static infrastructure addresses are reserved at the lower end; client pools start at `.50`.

| Site | VLAN | Purpose | Network | Gateway | DHCP pool / static allocation |
|---|---:|---|---|---|---|
| HQ | 10 | Staff | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.50-200` |
| HQ | 20 | Guest | `10.10.20.0/24` | `10.10.20.1` | `10.10.20.50-220` |
| HQ | 30 | Servers | `10.10.30.0/24` | `10.10.30.1` | Static only |
| HQ | 99 | Management | `10.10.99.0/24` | `10.10.99.1` | `10.10.99.50-100` or static |
| Branch | 10 | Staff | `10.20.10.0/24` | `10.20.10.1` | `10.20.10.50-200` |
| Branch | 20 | Guest | `10.20.20.0/24` | `10.20.20.1` | `10.20.20.50-220` |
| Branch | 30 | Servers | `10.20.30.0/24` | `10.20.30.1` | Static only |
| Branch | 99 | Management | `10.20.99.0/24` | `10.20.99.1` | `10.20.99.50-100` or static |

## Infrastructure and transit addresses

| Device/interface | Address | Notes |
|---|---|---|
| HQ-R1 G0/1 | `10.255.0.1/30` | Routed WAN to Branch |
| BR-R1 G0/1 | `10.255.0.2/30` | Routed WAN to HQ |
| pfSense LAN | `10.255.10.1/30` | Transit to HQ-R1 |
| HQ-R1 G0/2 | `10.255.10.2/30` | Default route next hop to pfSense |
| ISP/upstream router | `198.51.100.1/30` | Documentation-only example gateway |
| pfSense WAN | `198.51.100.2/30` | Replace with actual lab WAN configuration |
| OpenVPN client pool | `10.99.0.0/24` | Remote access tunnel addresses |

## HQ server reservations

| Host | Address | Gateway | Role |
|---|---|---|---|
| HQ-DHCP | `10.10.30.10/24` | `10.10.30.1` | Central DHCP service |
| HQ-DNS | `10.10.30.11/24` | `10.10.30.1` | Internal DNS service |
| HQ-WEB | `10.10.30.20/24` | `10.10.30.1` | Test intranet / VPN target |
| HQ-SW1 | `10.10.99.2/24` | `10.10.99.1` | Switch management SVI |
| BR-SW1 | `10.20.99.2/24` | `10.20.99.1` | Switch management SVI |

The branch VLAN 30 can hold static services such as a local print/file server, for example `10.20.30.10/24`, gateway `10.20.30.1`.

## Design cautions

- Do not reuse a subnet on both sites; OSPF requires unique routed prefixes.
- Avoid using `10.0.0.0/8` for a real enterprise if it overlaps with existing VPN/cloud networks. The broad route on pfSense is for this isolated lab only.
- `198.51.100.0/24` is reserved for documentation and examples. Do not expect it to route on the public Internet.
