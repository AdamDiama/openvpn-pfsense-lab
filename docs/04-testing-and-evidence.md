# 04: Testing and evidence

All tests were run from **Ubuntu-Remote**. The aim was to show three things: the office is unreachable without the VPN, a connected user can reach only what is explicitly allowed, and the tunnel traffic is encrypted.

## Test 1: Baseline (no VPN)

```bash
ping -c 3 192.168.1.50
```

Result: 100% packet loss. The office LAN is not reachable from the outside.

![Baseline](../images/01-before-vpn-ping-fails.png)

## Test 2: VPN connectivity

After connecting with OpenVPN, `tun0` was created with `10.10.10.2/24`, and both internal hosts answered:

```bash
ping -c 3 192.168.1.1     # pfSense LAN: replies
ping -c 3 192.168.1.50    # Ubuntu-LAN: replies
```

At this point the default "allow any" OpenVPN rule was still in place, so the user could reach everything.

![Connected](../images/04-ping-through-tunnel.png)

## Test 3: Least privilege

After restricting the OpenVPN rule to **TCP 80 → 192.168.1.50** (and resetting firewall states):

| Command | Expected | Result |
|---|---|---|
| `curl -m 5 http://192.168.1.50` | Allowed | Page returned |
| `ping -c 3 192.168.1.50` | Blocked (ICMP not allowed) | No replies |
| `curl -m 5 http://192.168.1.50:8080` | Blocked (wrong port) | Timed out |
| `ping -c 3 192.168.1.1` | Blocked (wrong destination) | No replies |

![Rule before](../images/06-firewall-rule-before.png)
![Rule after](../images/07-firewall-rule-after.png)
![Least privilege tests](../images/08-least-privilege-tests.png)

## Test 4: Proving the tunnel is encrypted

Packet captures were run in the pfSense shell (console option 8) while repeating `curl http://192.168.1.50` from Ubuntu-Remote.

**WAN side** (outside the firewall, where the tunnel travels):

```bash
tcpdump -i em0 -n -A -c 15 udp port 1194
```

The packets run between `10.0.2.4` and `10.0.2.3` on UDP 1194. The payload is not human-readable.

![WAN capture](../images/09-capture-wan-encrypted.png)

**LAN side** (inside the office, after the firewall has decrypted the traffic):

```bash
tcpdump -i em1 -n -A -c 20 tcp port 80
```

The request appears between `10.10.10.2` and `192.168.1.50` as readable HTTP.

![LAN capture](../images/10-capture-lan-readable.png)

**Conclusion:** the same web request is unreadable while crossing the "internet" and only appears in plain text once it is inside the trusted network. The tunnel is doing its job. (The test used plain HTTP deliberately so the difference would be visible; real traffic should also use HTTPS.)
