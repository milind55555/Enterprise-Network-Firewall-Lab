# Wi-Fi setup and segmentation

## SSID and VLAN plan

| Site | Access point role | SSID | Switch port VLAN | Security |
|---|---|---|---:|---|
| HQ | Staff AP | `HQ-STAFF` | 10 | WPA2-Personal/AES for lab |
| HQ | Guest AP | `HQ-GUEST` | 20 | WPA2-Personal/AES; rotate guest key |
| Branch | Staff AP | `BR-STAFF` | 10 | WPA2-Personal/AES for lab |
| Branch | Guest AP | `BR-GUEST` | 20 | WPA2-Personal/AES; rotate guest key |

In Packet Tracer, configure each AP's SSID and wireless security in its Config/GUI tab. Assign the AP's wired switch port to the matching access VLAN. Connect a wireless laptop to the matching SSID and obtain an IP address via DHCP.

## Why separate APs are used

A basic Packet Tracer AP may expose only one SSID and may not tag multiple VLANs. Using one AP per VLAN makes the mapping explicit and avoids pretending that a single untagged AP uplink is carrying multiple SSIDs/VLANs. In a real enterprise WLAN, use a VLAN-aware AP and wireless controller or a platform that maps each SSID to a tagged VLAN.

## Wi-Fi acceptance tests

- Staff wireless client at HQ receives `10.10.10.0/24`.
- Guest wireless client at HQ receives `10.10.20.0/24`.
- Staff wireless client at Branch receives `10.20.10.0/24`.
- Guest wireless client at Branch receives `10.20.20.0/24`.
- Guest cannot access internal private networks, but can reach permitted Internet destinations.
- Use unique credentials in the lab and do not publish them in screenshots or Git history.
