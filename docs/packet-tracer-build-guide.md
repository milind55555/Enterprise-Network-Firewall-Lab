# Packet Tracer build guide

## 1. Devices to place

Use Packet Tracer equivalents available in your version:

- 2 x Cisco 2911 routers (HQ-R1 and BR-R1; any router with enough Gigabit interfaces and subinterface support is acceptable).
- 2 x Cisco 2960 access switches (HQ-SW1 and BR-SW1).
- At least 2 PCs per site (one staff and one guest); add server endpoints at HQ.
- 2 access points per site: one for staff VLAN 10 and one for guest VLAN 20.
- 1 or more Server-PT devices at HQ for DHCP, DNS, and HTTP.

The pfSense portion is a separate VM; Packet Tracer does not emulate pfSense.

## 2. Cable the topology

| From | To | Link type |
|---|---|---|
| HQ-R1 G0/0 | HQ-SW1 G0/1 | Copper straight-through; 802.1Q trunk configured on switch |
| BR-R1 G0/0 | BR-SW1 G0/1 | Copper straight-through; 802.1Q trunk configured on switch |
| HQ-R1 G0/1 | BR-R1 G0/1 | Copper crossover or automatic cable; routed /30 WAN |
| HQ-R1 G0/2 | pfSense LAN NIC | Separate VM/virtual lab link; /30 transit |
| pfSense WAN NIC | ISP/upstream network | Separate VM/virtual lab link |
| PCs/APs/servers | Correct access switch ports | Copper straight-through |

If your selected router has different port numbering, adapt the configs and update this table.

## 3. Suggested switch port map

| Switch | Port(s) | VLAN | Connection |
|---|---|---:|---|
| HQ-SW1 | Gi0/1 | Trunk | HQ-R1 G0/0 |
| HQ-SW1 | Fa0/1-8 | 10 | HQ staff PCs / staff AP |
| HQ-SW1 | Fa0/9-12 | 20 | Guest AP / guest PCs |
| HQ-SW1 | Fa0/13-16 | 30 | DHCP, DNS, web servers |
| HQ-SW1 | Fa0/17-20 | 99 | Network management endpoints |
| BR-SW1 | Gi0/1 | Trunk | BR-R1 G0/0 |
| BR-SW1 | Fa0/1-8 | 10 | Branch staff PCs / staff AP |
| BR-SW1 | Fa0/9-12 | 20 | Guest AP / guest PCs |
| BR-SW1 | Fa0/13-16 | 30 | Branch server/printer endpoints |
| BR-SW1 | Fa0/17-20 | 99 | Network management endpoints |

The sample switch configs shut down Fa0/21-24 and Gi0/2. Change those ranges if you use those ports.

## 4. Apply the configurations

1. Open each switch CLI and paste the matching `configs/cisco/*switch.cfg` configuration in manageable sections.
2. Open each router CLI and paste the matching `configs/cisco/*router.cfg` configuration in manageable sections.
3. Verify VLANs/trunks before connecting all end devices.
4. Check router subinterfaces are up/up and the OSPF neighbor reaches FULL.
5. Configure HQ-DHCP, HQ-DNS, and HQ-WEB server services as described in `dhcp-dns-setup.md`.
6. Configure AP SSIDs and security as described in `wifi-setup.md`.

**Important:** Some Packet Tracer IOS images do not implement every IOS command exactly. If a command is rejected, check the device model/image and use the equivalent supported syntax. Do not paste all configuration blindly into a production device.

## 5. Configure client PCs

Set staff and guest PCs to DHCP. They should obtain:

- An IP address from the correct site/VLAN range.
- The VLAN gateway as default gateway.
- `10.10.30.11` as DNS server (including at the branch, reachable over OSPF).

If the Packet Tracer server's DHCP service does not support the exact options shown, create one pool per VLAN using the GUI and reserve the static addresses.

## 6. pfSense VM build

Create a pfSense CE/Plus VM with two virtual NICs on separate virtual networks. Assign WAN and LAN addresses from `docs/ip-addressing.md`, then follow the pfSense runbooks. A fully integrated lab is easiest in GNS3 or EVE-NG; in a Packet Tracer-only exercise, document the pfSense section as a separate companion VM and validate it independently.

## 7. Save and document

Save your Packet Tracer file locally, then place it under `packet-tracer/`. Add screenshots to `evidence/`. Confirm no credentials, VPN exports, or private certificates are present before publishing.
