# Lab 2 — A Windows guest, and a network between them

Pair: Imanbayeva Daiana — individual work   Driver: Imanbayeva Daiana (the whole lab)
Machine: Acer Nitro ANV15-41, Windows 11 Pro (10.0.26200), 16 GB RAM (15.3 GB usable), AMD Ryzen 7 7735HS (8 cores / 16 threads); guests Ubuntu Server 26.04.1 LTS (`lab01-ubuntu`) + Windows 11 Enterprise evaluation (`lab02-windows`), VirtualBox 7.2
Date: 08–09.11.2026

## What I did

1. **Downloaded Windows 11 Enterprise evaluation** (ISO, Enterprise, 64-bit, English (United States)). Licence: free 90-day evaluation, no product key — <https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise>. The ISO is kept in `Downloads`, not in this repository. When I picked it in the new-VM wizard, VirtualBox recognised it as *Windows 11 Enterprise Evaluation (x64 / en-US)* and switched the version from Windows 10 to Windows 11 (64-bit).
2. **Created `lab02-windows`: 2 vCPU, 4096 MB, 64 GB thin VDI, UEFI + Secure Boot, vTPM 2.0.** Unattended installation was left unticked, so I went through the installer myself. *Pre-allocate full size* was left unticked, so the disk file grows only as Windows writes to it. Choosing *Windows 11 (64-bit)* turned on EFI, Secure Boot and TPM 2.0; I checked them in *Settings → System* ([screenshots/vm-settings-system.png](screenshots/vm-settings-system.png)). Installed Windows with a local account, switched every privacy option off (diagnostic data: *Required only*), then installed VirtualBox Guest Additions 7.2.20 from *Devices → Insert Guest Additions CD image* → `VBoxWindowsAdditions.exe`.
3. **Network mode: NAT Network (`NatNetwork`, 10.0.2.0/24), both guests.** Why: the lab needs two things at once — the guests must reach **each other** and **the internet**.
   - Plain *NAT* (the VirtualBox default, and what `lab02-windows` had at first — [screenshots/network-plain-nat-before.png](screenshots/network-plain-nat-before.png)) gives every VM its own private router, so the two guests could never see each other.
   - *Host-only* lets them see each other but gives no internet.
   - *Bridged* would put both guests on the real Wi-Fi with their own MAC addresses; the campus network drops unknown MACs, and at home it would expose the guests to every device on the network. I did not need anyone outside my laptop to reach them.
   - NAT Network is the only mode that gives both: one shared private network for the two guests, and the internet through the host. Trade-off: the host cannot reach the guests directly without a port-forwarding rule.
4. **Ping Ubuntu → Windows timed out** (4 sent, 0 received, 100 % loss — [screenshots/ubuntu-to-windows-timeout.png](screenshots/ubuntu-to-windows-timeout.png)) **because** ping is an ICMP echo request, and Windows Defender Firewall blocks incoming echo requests by default. `Get-NetConnectionProfile` shows the guest's network is **Public**, the strictest profile ([evidence/ipconfig-windows.txt](evidence/ipconfig-windows.txt)). Windows → Ubuntu worked from the start, because Ubuntu's firewall (ufw) is off by default. Fixed with one rule that allows exactly ping, in PowerShell as Administrator — the firewall itself stayed on:
   ```powershell
   New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
   Get-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" | Format-List DisplayName,Enabled,Direction,Action,Profile
   ```
   The check returned `Enabled: True, Direction: Inbound, Action: Allow, Profile: Any` ([screenshots/firewall-rule-check.png](screenshots/firewall-rule-check.png)), and Ubuntu → Windows then got 4/4 replies ([screenshots/ubuntu-to-windows-after-rule.png](screenshots/ubuntu-to-windows-after-rule.png)).
5. **Overcommitment: set 10 GB (Ubuntu) + 10 GB (Windows) = 20 GB promised on a 16 GB host** (15.3 GB usable). Each VM alone stays below the host's total, only the sum exceeds it. Then I set the memory back (step 8 below).

All Ubuntu commands were run over SSH from PowerShell inside the Windows guest (`ssh vboxuser@10.0.2.4 "..."`), and the outputs were copied to the host through the shared clipboard (*Devices → Shared Clipboard → Bidirectional*). Running SSH from the Windows guest to the Ubuntu guest is one more proof that they are on one network.

## Network diagram

```
                        Internet
                           |
          Host laptop  Acer Nitro ANV15-41 (Wi-Fi)
          Windows 11 Pro, VirtualBox 7.2
                           |
      [ VirtualBox NAT Network "NatNetwork" - 10.0.2.0/24 ]
      [ NAT router / gateway 10.0.2.1                     ]
               /                              \
      lab01-ubuntu                         lab02-windows
      10.0.2.4/24 (DHCP)                   10.0.2.10/24 (static)
      Ubuntu Server 26.04.1 LTS            Windows 11 Enterprise eval
```

## Result

Host memory, from Task Manager → Performance → Memory:

| | Before (both VMs off) | After (Ubuntu 10 GB + Windows 10 GB, both running) |
|---|---|---|
| Host RAM in use | 10.0 GB of 15.3 GB (65 %) | 15.1 GB of 15.3 GB (99 %) |
| Available | 5.3 GB | 225 MB |
| Committed | 16.1 / 33.3 GB | 37.0 / 37.9 GB |
| Cached | 1.2 GB | 177 MB |
| What I noticed | — | Both VMs booted. Edge opened a website in Windows and `sudo apt update` ran in Ubuntu. I did not feel any stutter or freeze. |

What the numbers mean. The guests were *promised* 20 GB but the hypervisor does not take all of it at once: a guest only gets real host memory for the pages it actually touches. An idle Ubuntu Server uses about 200 MB (`free -m` in the 1 GB guest: 200 MB used), and Windows with one Edge window uses a few GB. That is why nothing froze. But the host had **no reserve left**: available memory fell from 5.3 GB to 225 MB, the file cache was squeezed from 1.2 GB to 177 MB, and *Committed* hit its limit — the limit itself grew from 33.3 to 37.9 GB because Windows enlarged the page file to cover the promises. One more heavy program on the host, or the guests starting to really use their 10 GB, and the host would have to swap to disk and slow down. Overcommitment does not fail immediately; it fails when the guests start using what they were promised.

**Step 8 — memory set back:** `lab02-windows` 4096 MB, `lab01-ubuntu` 1024 MB ([screenshots/memory-restored-windows-4096.png](screenshots/memory-restored-windows-4096.png), [screenshots/memory-restored-ubuntu-1024.png](screenshots/memory-restored-ubuntu-1024.png)). The calculation: with both VMs off the host has 5.3 GB available. Windows 11 cannot go below 4 GB (its minimum). In Lab 1 Ubuntu had 2 GB, but 4 + 2 = 6 GB is more than the 5.3 GB the host can give, which I had already predicted in Lab 1. Ubuntu Server has no graphical shell and used 200 MB of its memory at idle, so 1 GB is enough: 4 + 1 = 5 GB, which leaves about 0.3 GB for the host. That margin is small, so I close other programs on the host when both guests run.

## What did not work the first time

1. **Windows did not get a working address from the NAT Network.** After switching the adapter to NAT Network, Windows had no default gateway: `Remove-NetRoute ... -DestinationPrefix "0.0.0.0/0"` answered *"No matching MSFT_NetRoute objects found"*, so there was no route to anything outside the guest ([screenshots/windows-static-ip-fix.png](screenshots/windows-static-ip-fix.png)). Ubuntu on the same network did get its address by DHCP (`10.0.2.4/24 ... dynamic` in [evidence/ip-addr-linux.txt](evidence/ip-addr-linux.txt)), so the network itself worked. I gave Windows a fixed address in the same network instead, in PowerShell as Administrator:
   ```powershell
   Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled
   Remove-NetIPAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4 -Confirm:$false
   New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.2.10 -PrefixLength 24 -DefaultGateway 10.0.2.1
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8
   ```
   After that `ipconfig` showed 10.0.2.10 / 255.255.255.0 / gateway 10.0.2.1, and Windows reached Ubuntu, 8.8.8.8 and ubuntu.com ([screenshots/windows-ipconfig-ping-ubuntu.png](screenshots/windows-ipconfig-ping-ubuntu.png)). I did not save `ipconfig` from *before* the fix, so the route error above is the only record of the broken state.
2. **Ubuntu → Windows ping timed out** — the Windows firewall, see step 4 above.
3. **Ubuntu could ping 8.8.8.8 but not ubuntu.com**: `ping: ubuntu.com: Temporary failure in name resolution` ([screenshots/ubuntu-dns-failure.png](screenshots/ubuntu-dns-failure.png)). Pinging an IP address worked, so the network and the route to the internet were fine; only name resolution (DNS) was broken. `resolvectl status enp0s3` showed the DNS server Ubuntu got from DHCP, 192.168.100.1. After `sudo reboot` the DNS server was the same, and `ping -c 4 ubuntu.com` got 4/4 replies ([screenshots/ubuntu-dns-fixed-after-reboot.png](screenshots/ubuntu-dns-fixed-after-reboot.png)). So the server was fine and the guest was holding stale network state from before I switched it from NAT to NAT Network; the network is set up at boot, which is why the lab says to reboot after changing the network of a running Ubuntu VM.

## Evidence

- [evidence/ipconfig-windows.txt](evidence/ipconfig-windows.txt) — Windows address 10.0.2.10/24, gateway 10.0.2.1; network profile *Public* (the reason ping was blocked)
- [evidence/ip-addr-linux.txt](evidence/ip-addr-linux.txt) — Ubuntu address 10.0.2.4/24 on `enp0s3`, received by DHCP from the NAT Network
- [evidence/ping-both-ways.txt](evidence/ping-both-ways.txt) — Ubuntu → Windows, Ubuntu → ubuntu.com, Windows → Ubuntu, Windows → ubuntu.com, all 4/4 replies, unedited, with the command above each output
- [evidence/host-memory-before.png](evidence/host-memory-before.png) — host memory with both guests off: 10.0 GB in use, 5.3 GB available, 16.1 / 33.3 GB committed
- [evidence/host-memory-after.png](evidence/host-memory-after.png) — host memory with both guests at 10 GB and running: 15.1 GB in use (99 %), 225 MB available, 37.0 / 37.9 GB committed

Supporting screenshots (not required evidence) are in [screenshots/](screenshots/): VM system settings, the adapter before (NAT) and after (NAT Network), the failed and the working ping, the firewall rule, the DNS failure and fix, and the memory set back.
