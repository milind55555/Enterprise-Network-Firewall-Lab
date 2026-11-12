# Troubleshooting guide

## Router subinterface is down/down

- Confirm the physical router interface is `no shutdown`.
- Confirm the switch port connected to the router is a trunk.
- Confirm the VLAN exists and is allowed on the trunk.
- Check that the router subinterface VLAN ID matches the switch VLAN ID.

## OSPF neighbor does not reach FULL

- Check both WAN interfaces are up/up and use addresses in `10.255.0.0/30`.
- Check subnet masks match (`255.255.255.252`).
- Ensure the WAN interface is not passive in OSPF.
- Check area ID, authentication settings (if added), and ACLs.
- Use `show ip ospf neighbor`, `show ip protocols`, and `show ip route`.

## Branch DHCP fails

- Confirm the HQ DHCP server is static at `10.10.30.10` and gateway is `10.10.30.1`.
- Confirm branch router VLAN subinterface has `ip helper-address 10.10.30.10`.
- Confirm OSPF routes exist between Branch and HQ server VLAN.
- Confirm DHCP pool network, default gateway, DNS server, and available range are correct.
- Ensure no ACL blocks the relayed DHCP server traffic on the routed path.

## Guest can reach internal networks

- Confirm `ip access-group HQ-GUEST-IN in` or `BR-GUEST-IN in` is applied to the correct guest subinterface.
- Confirm the ACL deny entries appear before the final permit.
- Confirm the ACL source network matches the guest subnet.
- Check counters with `show access-lists`.
- Check that you are testing a private destination and not a public service.

## Guests cannot resolve DNS

- Confirm guest ACL has DNS permit entries before the RFC1918 deny entries.
- Confirm DHCP advertises `10.10.30.11` as DNS.
- Confirm the DNS service is running and the DNS server's default gateway is `10.10.30.1`.

## Staff Internet fails through pfSense

- Check HQ-R1 default route points to `10.255.10.1`.
- Check Branch learned the default route from OSPF.
- Check pfSense has a WAN gateway and a static route for office networks via `10.255.10.2`.
- Add/verify the outbound NAT rule for routed source range `10.0.0.0/8`.
- Verify LAN rules permit the required egress ports.
- Inspect firewall logs and states before widening rules.

## OpenVPN connects but cannot reach intranet

- Check client receives `10.99.0.x`.
- Check pushed local networks include `10.10.30.0/24`.
- Check pfSense static route to office networks via `10.255.10.2`.
- Check HQ-R1 route to `10.99.0.0/24` via `10.255.10.1`.
- Check OpenVPN firewall rule allows the exact host and port.
- Check host firewall on HQ-WEB permits the test service.
- Confirm DNS is not the only failing part by testing the IP directly.
