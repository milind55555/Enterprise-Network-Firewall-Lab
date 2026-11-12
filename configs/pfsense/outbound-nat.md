# pfSense outbound NAT for routed VLANs

## Why a manual rule is required

The pfSense LAN interface is only the transit network `10.255.10.0/30`. User VLANs (`10.10.x.0/24` and `10.20.x.0/24`) sit behind HQ-R1 and are not directly connected to pfSense. The default automatic NAT rules may not include these downstream prefixes. Add an explicit outbound NAT rule.

## Configuration

1. Open **Firewall > NAT > Outbound**.
2. Select **Hybrid Outbound NAT** (or Manual mode if you want to own every rule).
3. Save/apply.
4. Add a rule:
   - Interface: `WAN`
   - Address Family: `IPv4`
   - Protocol: `Any`
   - Source: `10.0.0.0/8`
   - Source port: `Any`
   - Destination: `Any`
   - Destination port: `Any`
   - Translation / target: `Interface address`
   - Description: `NAT_OFFICE_AND_VPN_RFC1918_TO_WAN`
5. Apply changes.

The `/8` source covers the lab's HQ, branch, and VPN networks. For a production environment, replace it with the explicit approved prefixes and check that no unrelated private networks are unintentionally translated.

## Validation

- From an HQ staff client, browse to an external test endpoint or ping a permitted public IP if ICMP egress is allowed.
- Repeat from a branch staff client.
- On pfSense, inspect **Firewall > NAT > Outbound** and **Status > System Logs > Firewall**.
- Confirm the public-side connection is translated to the pfSense WAN address.

Do not NAT traffic between internal VLANs; that traffic is routed by Cisco routers and should retain its original source addresses.
