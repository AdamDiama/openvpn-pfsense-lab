# 03: Ubuntu-Remote (the remote worker)

This VM represents an employee working from outside the office. It sits on the simulated internet (`WANNET`) and has no direct route to the office LAN.

## Creating the VM

Cloned from Ubuntu-LAN (VirtualBox → Clone, with **Generate new MAC addresses for all network adapters**). Then:

1. **Settings → Network → Adapter 1:** NAT Network → `WANNET`
2. The clone inherited the static LAN address, so it was set back to automatic:

```bash
sudo nmcli connection modify "netplan-enp0s3" \
  ipv4.method auto ipv4.addresses "" ipv4.gateway "" ipv4.dns ""
sudo nmcli connection up "netplan-enp0s3"
ip a        # enp0s3 now shows a 10.0.2.x address (10.0.2.4 in this lab)
```

Note: pings to the pfSense WAN address (`10.0.2.3`) fail by design. The firewall drops unsolicited traffic on the WAN, except the VPN port. See troubleshooting.

## Install the OpenVPN client

```bash
sudo apt update && sudo apt install -y openvpn
openvpn --version
```

## Before connecting (baseline)

```bash
ping -c 3 192.168.1.50      # fails: the office is not reachable from outside
```

![Before VPN](../images/01-before-vpn-ping-fails.png)

## Connect

The exported `.ovpn` file was copied into `~/Downloads`:

```bash
cd ~/Downloads
sudo openvpn --config pfSense-UDP4-1194-labuser-config.ovpn
```

Log in as `labuser`. The connection is up when the log shows:

```
Initialization Sequence Completed
```

Keep this terminal open (closing it ends the tunnel) and use a second terminal for tests. The tunnel address is `10.10.10.2/24` on `tun0`.

![Connected](../images/02-openvpn-initialization-complete.png)

## Verify

```bash
ip a                        # tun0 with 10.10.10.2/24
ping -c 3 192.168.1.1       # pfSense LAN
ping -c 3 192.168.1.50      # Ubuntu-LAN
```

![tun0](../images/03-tun0-interface.png)
![Pings through the tunnel](../images/04-ping-through-tunnel.png)
![pfSense shows labuser connected](../images/05-pfsense-openvpn-status.png)

To stop the VPN, press `Ctrl + C` in the OpenVPN terminal.
