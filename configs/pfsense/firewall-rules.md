# pfSense interface and firewall rules

This runbook assumes pfSense is a separate VM between an ISP/upstream network and HQ-R1.

## Interface addressing

| pfSense interface | Address | Gateway / note |
|---|---|---|
| WAN | `198.51.100.2/30` | Upstream gateway `198.51.100.1` (documentation example only) |
| LAN | `10.255.10.1/30` | HQ-R1 transit peer `10.255.10.2` |
| OpenVPN | `10.99.0.0/24` tunnel pool | Created by OpenVPN server configuration |

In a VM, map the WAN and LAN virtual NICs to separate virtual networks. Do not connect both interfaces to the same layer-2 segment. Replace the example WAN range with addresses from your own lab network.

## Add a route back to the office networks

1. Open **System > Routing > Gateways** and confirm the WAN gateway.
2. Open **System > Routing > Static Routes** and add:
   - Destination network: `10.0.0.0/8`
   - Gateway: the LAN-side next hop `10.255.10.2` (create/select a gateway on the LAN interface if the UI requires it).
   - Description: `ROUTE_TO_OFFICE_NETWORKS_VIA_HQ_R1`
3. Save and apply. Verify that the route appears in the routing table.

This broad route is convenient for the isolated lab. A production design should use only the exact HQ and branch prefixes.

## Firewall policy overview

pfSense is stateful: return traffic for permitted connections is automatically allowed. Rules are evaluated top-to-bottom on the interface where traffic enters. The Cisco guest ACLs provide internal guest isolation before traffic reaches the perimeter firewall.

**Important default-rule step:** A new pfSense installation commonly has a `Default allow LAN to any` rule whose source is the directly connected LAN subnet. In this design, that subnet is only the /30 transit link, so it does not automatically match the routed staff/guest VLAN source addresses. Still, disable/remove the broad rule after adding the trusted-admin management rule so the intended policy is explicit and safe if you later change the LAN mask. Keep console access available so you can recover if you lock yourself out. Do not remove the platform's anti-lockout protection unless you fully understand the consequences.

### WAN interface

- Keep the default deny behavior for unsolicited inbound traffic.
- Permit only the OpenVPN server listener to **This Firewall**, UDP destination port `1194` (or the port selected in your OpenVPN server).
- Do not create broad WAN allow rules such as `any -> any`.
- If pfSense is behind another NAT router, forward only the selected OpenVPN UDP port from that upstream device to pfSense WAN.

### LAN interface (traffic from HQ-R1 toward pfSense)

Create rules in this order, adjusting aliases if desired. The first rule is only for a trusted admin workstation that needs to manage the firewall GUI/SSH; change the example source to your actual admin host or management subnet.

| Order | Action | Protocol | Source | Destination | Destination port | Purpose |
|---:|---|---|---|---|---|---|
| 1 | Pass | TCP | `10.10.99.50` (trusted admin host example) | This Firewall | 443 (and 22 only if SSH management is enabled) | Manage pfSense from a trusted admin endpoint |
| 2 | Pass | UDP/TCP | `10.10.30.11` | any | 53 | Allow internal DNS resolver to query upstream DNS |
| 3 | Pass | UDP | `10.0.0.0/8` | any | 123 | NTP for office networks |
| 4 | Pass | TCP | `10.0.0.0/8` | any | 80, 443 | Standard web egress |
| 5 | Pass (optional) | ICMP | `10.0.0.0/8` | any | any | Diagnostics; remove after testing if not needed |
| 6 | Block/log | any | any | any | any | Explicit deny for unmatched traffic |

For a learning lab, you may temporarily add a broader `10.0.0.0/8 -> any` pass rule to troubleshoot. Remove it after confirming connectivity and document the final least-privilege policy. If you do not have an admin host in VLAN 99, use an approved admin host address from the subnet you actually use. The source example is not a requirement to deploy another device.

## VPN interface rules

On **Firewall > Rules > OpenVPN**, permit only approved destinations and ports. Example split-tunnel policy:

| Order | Action | Protocol | Source | Destination | Port | Purpose |
|---:|---|---|---|---|---|---|
| 1 | Pass | TCP | `10.99.0.0/24` | `10.10.30.20` | 443 | Intranet HTTPS |
| 2 | Pass (optional) | ICMP | `10.99.0.0/24` | `10.10.30.20` | any | Ping for troubleshooting |
| 3 | Block/log | any | `10.99.0.0/24` | `10.0.0.0/8` | any | Block other private-network access |

Do not add a blanket VPN-to-any rule if the objective is restricted remote access. Add specific ports and hosts required by your test services. Branch networks are not included in this baseline policy.

## Rule verification

- Check **Status > System Logs > Firewall** while testing.
- Confirm a VPN client can reach `10.10.30.20:443` after connecting.
- Confirm a VPN client cannot reach an unapproved host or port.
- Confirm guest ACL counters increase when a guest tries to reach private networks.
