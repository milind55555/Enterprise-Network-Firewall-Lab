# Security policy and control matrix

| Source | Destination | Expected behavior | Enforcement |
|---|---|---|---|
| HQ staff VLAN 10 | HQ servers VLAN 30 | Permit required business services | HQ router inter-VLAN routing; pfSense egress policy for Internet |
| Branch staff VLAN 10 | HQ servers VLAN 30 | Permit required business services | OSPF-routed path via BR-R1/HQ-R1 |
| HQ guest VLAN 20 | HQ/Branch private networks | Deny, except DNS to `10.10.30.11` and DHCP relay | `HQ-GUEST-IN` extended ACL |
| Branch guest VLAN 20 | HQ/Branch private networks | Deny, except DNS to `10.10.30.11` and DHCP relay | `BR-GUEST-IN` extended ACL |
| Guest VLANs | Public destinations | Permit after private destinations are denied | Cisco guest ACL + pfSense egress rules/NAT |
| Internet | Internal networks | Deny unsolicited inbound traffic; allow VPN listener only | pfSense WAN default-deny policy |
| VPN client pool | HQ intranet `10.10.30.20:443` | Permit | pfSense OpenVPN rules |
| VPN client pool | Other private networks | Deny by default / explicit block | pfSense OpenVPN rules |
| Admin workstation | Cisco device management | SSH only from approved management VLAN | Cisco VTY SSH-only settings; production should add source ACL |

## Notes and limitations

- Cisco guest ACLs block RFC1918 destinations but allow public destinations. If your organization uses public IP space internally, add those prefixes to the deny list.
- The sample pfSense rules are a learning baseline, not a complete production policy. Build explicit aliases, logging, egress allowlists, anti-lockout access, and change control for a real deployment.
- The baseline uses a shared lab secret placeholder. Real deployments require per-user identities, strong secrets, centralized authentication, logging, backups, and a formal key lifecycle.
- VLANs are a segmentation mechanism, not a substitute for identity-aware access control or endpoint security.
