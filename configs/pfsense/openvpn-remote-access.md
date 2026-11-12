# pfSense OpenVPN remote-access setup

This is a split-tunnel example for remote administrators/staff who need a narrow path to an HQ intranet service. UI labels can differ slightly by pfSense release.

## Prerequisites

- pfSense WAN is reachable from the test remote client.
- WAN inbound firewall permits UDP `1194` to the pfSense OpenVPN server only.
- pfSense static route `10.0.0.0/8` points to HQ-R1 `10.255.10.2`.
- HQ-R1 has `ip route 10.99.0.0 255.255.255.0 10.255.10.1`.
- The HQ test web server `10.10.30.20` has a static address and is reachable from HQ staff VLAN.

## Create the server

1. In pfSense, install/enable the OpenVPN Client Export package if required by your release.
2. Open **VPN > OpenVPN > Wizards** (or the server configuration page).
3. Create an internal Certificate Authority and a server certificate. Use unique, lab-only names.
4. Select **Remote Access (SSL/TLS + User Auth)** or the equivalent user-authenticated server mode.
5. Protocol: UDP over IPv4. Port: `1194`.
6. Interface: WAN.
7. Tunnel network: `10.99.0.0/24`.
8. Local network(s) to route through the tunnel: `10.10.10.0/24,10.10.30.0/24`.
9. DNS server: `10.10.30.11` if the remote user should resolve the lab's internal hostnames.
10. Use modern TLS/cipher defaults supported by your pfSense version; do not copy legacy cipher settings from old tutorials.
11. Create individual VPN users and certificates. Do not share one certificate among all users.
12. Save and apply.

## Firewall permissions

- WAN: pass UDP destination port `1194` to **This Firewall**.
- OpenVPN tab: permit TCP from `10.99.0.0/24` to `10.10.30.20` on port `443`.
- Optionally permit ICMP to `10.10.30.20` during troubleshooting.
- Block/log remaining VPN access to `10.0.0.0/8` so the tunnel cannot reach unapproved internal destinations.

Keep the VPN policy narrow. Do not add the guest subnets to the pushed local networks.

## Export and connect

1. Install/use the OpenVPN Client Export utility if it is separate in your pfSense release.
2. Export a profile for the individual VPN user and their client platform.
3. Transfer it securely to the test client. Do not commit `.ovpn`, private keys, or certificates to GitHub.
4. Connect and verify the assigned tunnel address is in `10.99.0.0/24`.
5. Test `https://10.10.30.20` (or the hostname resolved by HQ-DNS).
6. Test an unapproved internal IP/port and confirm it is blocked.

## Security checklist

- Use a unique account and certificate per person/device.
- Require strong user passwords and protect private keys.
- Revoke lost/retired client certificates.
- Restrict OpenVPN firewall rules to required hosts/ports.
- Keep pfSense and packages patched.
- Never publish exported profiles, certificates, private keys, or authentication secrets.
