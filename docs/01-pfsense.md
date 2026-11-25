# 01: pfSense (firewall and OpenVPN server)

pfSense acts as the office router/firewall. It has two network adapters: one facing the simulated internet (WAN) and one facing the office network (LAN).

## 1. VirtualBox networks

| Name | Type | Purpose |
|---|---|---|
| `WANNET` | NAT Network (File → Tools → Network Manager → NAT Networks) | Simulated internet, `10.0.2.0/24`, DHCP on |
| `LAN1` | Internal Network (chosen per VM adapter) | Office LAN |

## 2. VM settings

- Type: BSD, Version: FreeBSD (64-bit)
- 1 CPU, 1024 MB RAM, 16 GB disk
- Adapter 1: NAT Network → `WANNET`
- Adapter 2: Internal Network → `LAN1`

## 3. Install and interface assignment

Installed from the pfSense CE ISO using the guided installer (Auto UFS, entire virtual disk). After install, the console menu was used:

1. Option **1** (Assign Interfaces): VLANs = no, WAN = `em0`, LAN = `em1`
2. Option **2** (Set interface IP address) for LAN:
   - IPv4 `192.168.1.1/24`, no upstream gateway
   - DHCP server enabled, range `192.168.1.100` to `192.168.1.200`

Result:

```
WAN (wan) -> em0 -> v4/DHCP4: 10.0.2.3/24
LAN (lan) -> em1 -> v4: 192.168.1.1/24
```

## 4. Web interface and setup wizard

From a machine on the LAN: `https://192.168.1.1`. The default admin password was changed during the wizard.

**WAN block rules:** because the lab's "internet" uses private address space, I unticked both of these under **Interfaces → WAN** so the VPN traffic is not dropped:
- Block private networks and loopback addresses
- Block bogon networks

(On a real internet-facing WAN these stay enabled.)

## 5. DNS

Under **System → General Setup**, public DNS servers (`8.8.8.8`, `1.1.1.1`) were set so pfSense could resolve names and download packages. See the troubleshooting notes.

## 6. Install the client export package

**System → Package Manager → Available Packages → `openvpn-client-export`**

## 7. OpenVPN server (wizard: VPN → OpenVPN → Wizards)

| Setting | Value |
|---|---|
| Server type | Local User Access |
| Certificate Authority | `LabCA` (created in the wizard) |
| Server certificate | `LabServerCert` |
| Interface | WAN |
| Protocol / Port | UDP on IPv4 only / `1194` |
| Tunnel network | `10.10.10.0/24` |
| Local network | `192.168.1.0/24` |
| Concurrent connections | 2 |
| Firewall rules | "Firewall Rule" and "OpenVPN rule" ticked (auto-created) |

## 8. VPN user

**System → User Manager → Add**
- Username `labuser`, strong password
- Ticked "Click to create a user certificate" → `labuser-cert`, signed by `LabCA`

## 9. Export the client configuration

**VPN → OpenVPN → Client Export**
- Host Name Resolution: **Other**, Host Name: `10.0.2.3` (the pfSense WAN address)
- Under `labuser`, **Inline Configurations → Most Clients** downloads the `.ovpn` file

> Do not commit this file. It contains the client's private key.

## 10. Least-privilege firewall rule

By default the wizard creates an "allow any" rule on the **OpenVPN** tab, so a connected user can reach everything. I changed that rule (Firewall → Rules → OpenVPN):

| Field | Before | After |
|---|---|---|
| Protocol | Any | TCP |
| Destination | Any | Single host `192.168.1.50` |
| Destination port | Any | HTTP (80) |
| Description | (wizard default) | `Allow VPN users to web server only` |

Everything else falls through to pfSense's default deny. After **Apply Changes**, existing connection states were cleared under **Diagnostics → States → Reset States**, since pfSense otherwise keeps already-established connections alive.

The WAN tab keeps the rule created by the wizard that allows UDP `1194` to the firewall. That is what lets the VPN connect at all.

![Rule before](../images/06-firewall-rule-before.png)
![Rule after](../images/07-firewall-rule-after.png)
