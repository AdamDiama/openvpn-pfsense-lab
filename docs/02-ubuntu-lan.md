# 02: Ubuntu-LAN (the internal "office" machine)

This VM represents a server inside the office network. It hosts simple web servers used as targets for the access-control tests.

## VM settings

- Ubuntu Desktop 26.04, 2 CPUs, 2 to 4 GB RAM, 25 GB disk
- Adapter 1: **Internal Network → `LAN1`** (same name as pfSense adapter 2)
- Graphics Controller **VBoxSVGA**, 3D acceleration off (see troubleshooting)

Always start pfSense first, so it is ready to serve the network before the clients boot.

## Static address

A fixed address keeps the test target predictable. The connection is named `netplan-enp0s3` (check with `nmcli connection show`):

```bash
sudo nmcli connection modify "netplan-enp0s3" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.50/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns 192.168.1.1
sudo nmcli connection up "netplan-enp0s3"
```

Verify and test the path to the firewall:

```bash
ip a                      # enp0s3 should show 192.168.1.50/24
ping -c 3 192.168.1.1
```

From this machine, the pfSense web interface loads at `https://192.168.1.1` (accept the self-signed certificate warning).

## Test web servers

This VM has no internet, so rather than installing Apache I used Python's built-in web server. Two instances run in separate terminals:

```bash
# Allowed by the firewall rule
sudo python3 -m http.server 80

# Deliberately not allowed, used to prove the rule blocks other ports
python3 -m http.server 8080
```
