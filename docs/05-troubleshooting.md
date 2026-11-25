# 05: Troubleshooting log

Problems I hit while building the lab, how I diagnosed them, and what I took from each.

---

## 1. pfSense would not boot after install (`vm_fault: pager read error`)

**Symptom:** After installation, the console filled with `vm_fault: pager read error, pid 1 (init)` and the VM was stuck.

**Cause:** I detached the virtual install disc while the installer environment was still running from it, so the system could no longer read its own files. The installation on the virtual hard disk was already complete and unaffected.

**Fix:** Powered the VM off, confirmed the optical drive was empty and the hard disk was first in the boot order, then started it again. It booted to the normal pfSense console.

**Lesson:** Halt or power off before removing install media, then detach it. Check the boot order afterwards.

---

## 2. Ubuntu installer failed to start (`vmwgfx` graphics errors)

**Symptom:** The Ubuntu installer showed `vmwgfx seems to be running on an unsupported hypervisor` and did not load.

**Cause:** The VM's virtual graphics controller was not compatible with Ubuntu's graphics driver under VirtualBox.

**Fix:** Powered off, then in **Settings → Display** set the Graphics Controller to **VBoxSVGA**, video memory to 128 MB, and turned 3D acceleration off.

**Lesson:** Hypervisor graphics settings can break a guest OS even when the OS itself is fine.

---

## 3. Client had no address (`169.254.x.x`)

**Symptom:** Ubuntu-LAN could not load the pfSense page, and `ip a` showed a `169.254.x.x` link-local address.

**Cause:** The Ubuntu VM had started before pfSense was running, so its DHCP request got no reply, and it fell back to a self-assigned address.

**Fix:** Gave Ubuntu-LAN a static address with `nmcli` (`192.168.1.50/24`, gateway and DNS `192.168.1.1`). I also adopted the rule of starting pfSense first and waiting for its menu before booting clients.

**Lesson:** `169.254.x.x` means "no DHCP answer". Check the DHCP server, boot order and the cable and network assignment, in that order.

---

## 4. Package install hung on pfSense

**Symptom:** Installing `openvpn-client-export` sat on "sending request" with no progress.

**Diagnosis:** Used **Diagnostics → Ping** from the WAN to test reachability by IP and by name. Pinging an IP address worked, but names did not resolve, which pointed at DNS.

**Fix:** Added public DNS servers under **System → General Setup**, disabled DHCP overriding of the DNS list, then reran the install, which completed.

**Lesson:** "Can reach the internet by IP but not by name" is a DNS problem. Test both separately.

---

## 5. VM cloning failed with `VERR_DISK_FULL`

**Symptom:** Cloning Ubuntu-LAN to create the remote client stopped with `VERR_DISK_FULL`. The half-copied clone then refused to boot with "Could not read from the boot medium".

**Cause:** The host drive ran out of space during the copy, leaving an incomplete virtual disk.

**Fix:** Deleted the incomplete clone, freed up space on the host drive, and created the clone again from a fully powered-off source with new MAC addresses generated.

**Lesson:** A clone that will not boot is often an incomplete copy. Check the host's free space and the clone's disk before changing VM settings.

---

## 6. Ping to the pfSense WAN address failed (by design)

**Symptom:** From Ubuntu-Remote, `ping 10.0.2.3` had 100% packet loss even though the VM had a valid address on the same network.

**Cause:** pfSense blocks unsolicited inbound traffic on the WAN by default, including ICMP. Only the VPN port (UDP 1194) is allowed in.

**Resolution:** No fix needed. This is correct firewall behaviour. I used the VPN connection itself as the real connectivity test. If a ping test is wanted for troubleshooting, an ICMP echo rule can be added on the WAN and removed afterwards.

**Lesson:** A failed ping does not always mean a broken network. It can mean the firewall is doing its job.

---

## 7. Firewall changes not taking effect

**Note:** After restricting the OpenVPN rule, pfSense keeps existing connection states, so old traffic can continue to flow under the old rule. The change only applies cleanly after **Apply Changes** and **Diagnostics → States → Reset States**, then reconnecting the client. I did this before running the least-privilege tests.
