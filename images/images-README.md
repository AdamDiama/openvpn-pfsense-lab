# Screenshots



| File | What it shows |
|---|---|
| `01-before-vpn-ping-fails.png` | Ubuntu-Remote pinging 192.168.1.50 before connecting (fails) |
| `02-openvpn-initialization-complete.png` | OpenVPN log ending in "Initialization Sequence Completed" |
| `03-tun0-interface.png` | `ip a` showing `tun0` with 10.10.10.2/24 |
| `04-ping-through-tunnel.png` | Successful pings to 192.168.1.1 and 192.168.1.50 |
| `05-pfsense-openvpn-status.png` | pfSense Status → OpenVPN showing `labuser` connected |
| `06-firewall-rule-before.png` | OpenVPN rule with Protocol Any, Destination Any |
| `07-firewall-rule-after.png` | The restricted rule (TCP, 192.168.1.50, port 80) |
| `08-least-privilege-tests.png` | The four `curl`/`ping` tests after the rule |
| `09-capture-wan-encrypted.png` | `tcpdump` on em0, unreadable payload |
| `10-capture-lan-readable.png` | `tcpdump` on em1, readable HTTP |

