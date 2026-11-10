# Ticket 1 — New developer starts Monday, needs a test machine

Author: Imanbayeva Daiana · individual work · Date: 09.11.2026
Deliverable: [BUILD.md](BUILD.md) — a build document Aigerim can follow to rebuild both test machines from nothing.

## 1. What was asked

> **From:** Aigerim T. (Office Manager)
> **Subject:** test machine for new dev
>
> Hi, we have someone starting on Monday and he needs a test computer set up. Nothing special, just so he can try things without breaking anything. Can you write down how to do it so I can do the next one myself? Thanks

## 2. What was actually needed

**What the request left out, and what I assumed instead**

| The request does not say | What I would ask Aigerim | What I assumed |
|---|---|---|
| Which operating system the developer needs. "A test computer" names no OS. | "Which systems will he develop or test for — Linux servers, Windows desktops, or both?" | Both. A developer usually deploys to a Linux server and has to check how things work for Windows users. So: **two virtual machines**, Ubuntu Server 26.04.1 LTS and Windows 11 Enterprise evaluation. |
| What "breaking anything" means. | "What must stay safe — the office laptop, the office network, or his test setup?" | Two different things can break. (1) **The host laptop**: a virtual machine keeps his experiments inside a file, so the laptop and its data are not touched. (2) **The test machine itself**: he *will* break it — that is what testing is. So the real answer is a **clean snapshot** taken right after installation and a rollback that Aigerim can do in a minute. |
| Which computer it runs on, how much memory and disk it has. | "Which laptop will he use? How much RAM and free disk does it have?" | It is a laptop like mine: 16 GB RAM, of which Windows already uses about 10 GB at idle. The numbers in BUILD.md are calculated for that, and BUILD.md tells Aigerim how to check her own laptop before she starts (Step 0). |
| Whether the test machine needs the internet, and whether the two systems must see each other. | "Does he need to install packages from the internet? Does his Windows test machine need to talk to his Linux server?" | Yes to both — a developer installs packages, and a client on Windows has to reach a service on Linux. That decides the network mode: **NAT Network** (both guests on one private network, internet through the host). |
| Whether he needs to reach the test machines from other computers. | "Will anyone else connect to his test machines?" | No. Only from his own laptop. So the guests stay hidden behind the laptop (NAT), not bridged onto the office network. |
| Which software, which licences, whether it costs money. | "Does the office have Windows licences for test machines?" | Everything must be free and legal for an office: VirtualBox base package (GPLv3, free also for commercial use), Ubuntu (free), Windows 11 Enterprise **evaluation** (free for 90 days, no key). |
| Who the reader is. | — | Aigerim, an office manager, not an IT person. Every step in BUILD.md is one action, with a screenshot, why this choice, and how she knows it worked. |

**Where the request itself is wrong**

1. **"Nothing special" is not true.** Windows 11 refuses to install without UEFI, Secure Boot, a TPM 2.0 chip, 4 GB of memory and a 64 GB disk. A virtual machine has none of that unless you turn it on. If Aigerim believes it is nothing special, the first thing she will see is *"This PC can't run Windows 11"*.
2. **Memory is the real limit, not "a test computer".** On a 16 GB laptop Windows itself uses about 10 GB, so only about 5.3 GB is free. Windows 11 needs at least 4 GB and the default Linux size would be 2 GB — together 6 GB, which does not fit. The plan only works with Ubuntu reduced to 1 GB. On an 8 GB laptop both guests cannot run at the same time at all.
3. **"Without breaking anything" asks for the wrong thing.** A test machine that cannot break is useless for testing. What he needs is that a break costs nothing: the snapshot and the rollback are the most important part of the document, not an extra.
4. **"So I can do the next one myself" is limited in time.** The Windows evaluation stops working after 90 days. For the next developer Aigerim can simply build a fresh one from the same document, but if a test machine has to live longer than 90 days the office needs a real Windows licence. This is written at the top of BUILD.md.
5. **"The next one" will be on a different laptop.** Fixed numbers would be wrong there, so BUILD.md includes the check (Task Manager → Memory → *Available*) and the rule for choosing memory instead of only my numbers.

## 3. What I did

I rebuilt nothing from scratch for this ticket: both guests were built in Lab 1 (`lab01-ubuntu`) and Lab 2 (`lab02-windows`) on my laptop, and I wrote the steps down so that someone who has never opened a hypervisor can repeat them. Commands I ran while building and checking:

```powershell
# Host: hardware numbers (Lab 0)
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors
Get-CimInstance Win32_PhysicalMemory | Select-Object Manufacturer, @{n='GB';e={$_.Capacity/1GB}}, Speed
Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter, @{n='SizeGB';e={[math]::Round($_.Size/1GB,1)}}, @{n='FreeGB';e={[math]::Round($_.SizeRemaining/1GB,1)}}
```

```bash
# Ubuntu guest: SSH was missing after the unattended install
sudo apt install -y openssh-server
ip -br a
resolvectl status enp0s3
```

```powershell
# Windows guest, PowerShell as Administrator: address, firewall rule for ping, checks
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled
Remove-NetIPAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.2.10 -PrefixLength 24 -DefaultGateway 10.0.2.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8
New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
Get-NetConnectionProfile
ssh vboxuser@10.0.2.4 "ping -c 4 10.0.2.10; ping -c 4 ubuntu.com"
ping 10.0.2.4
ping ubuntu.com
```

```powershell
# Host: snapshot (Lab 1); the Windows snapshot was taken the same way from the Snapshots tab
VBoxManage snapshot "lab01-ubuntu" take clean-install --description "fresh install + openssh-server, nothing else changed"
```

### Decisions and the alternatives I rejected

| Decision | Alternative | Why I chose this — the trade-off |
|---|---|---|
| **VirtualBox** as the hypervisor | VMware Workstation Pro; Hyper-V built into Windows | VirtualBox is free for an office and runs on any Windows laptop, and Aigerim can install it from one website. VMware Workstation needs an account to download and has to encrypt the VM to add a TPM, which means one more password she could lose. Hyper-V needs Windows Pro/Enterprise and is turned on differently on every laptop. The price: on my laptop Hyper-V is already running, so VirtualBox runs on top of it and is a little slower. |
| **No VirtualBox Extension Pack** | Install it for USB 3 and disk encryption | The Extension Pack is under a different licence (PUEL) that is not free for commercial use. Nothing in this setup needs it, so leaving it out keeps the office from buying anything by mistake. |
| **Windows 11 Enterprise evaluation** | A Windows key from the office, or a "pre-activated" image | The evaluation is free and legal with no key. A pre-activated image is pirated software. The price: it expires in 90 days (see section 2). |
| **Ubuntu Server LTS** without a desktop | Ubuntu Desktop | Server needs about 200 MB at idle instead of 1–2 GB for a desktop, and that is the only way both guests fit into the laptop's free memory. LTS gets updates for 5 years. The price: he works in a terminal. |
| **NAT Network** for both guests | NAT, Bridged, Host-only | Plain NAT gives each VM its own router, so the two guests cannot see each other. Host-only has no internet. Bridged puts the guests on the office/campus network with their own MAC addresses — the campus network drops unknown MACs, and it would expose test machines to the whole office. NAT Network gives both things he needs. The price: the host cannot reach the guests without a port-forwarding rule. |
| **Firewall rule for ping**, firewall stays on | Turn the Windows firewall off | Turning it off makes ping work and leaves the whole machine open. One rule allows exactly ICMP echo and nothing else. |
| **Fixed address 10.0.2.10 for Windows** | Keep DHCP | Windows did not receive a gateway from the NAT Network's DHCP (section "One thing that went wrong"). A fixed address made it work and never changes, so the developer always knows where Windows is. The price: if a third VM is added, it must not be given 10.0.2.10. |
| **Ubuntu: unattended install**, Windows: manual install | Manual for both / unattended for both | For Ubuntu the unattended install is the simplest path for a non-IT reader: no partitioning questions. Its price: no SSH server, so BUILD.md adds one command. For Windows I went through the installer by hand, because the Windows unattended install hides the account and privacy choices that Aigerim should see and decide. |
| **Thin (dynamically allocated) disks** | Pre-allocate full size | Thin disks start small and grow only as the guest writes, so 64 + 25 GB does not take 89 GB on day one. The price: a little slower writes, which does not matter for a test machine. |

### Every setting that differs from the default, and why

| Setting | Default | Mine | Why |
|---|---|---|---|
| Ubuntu memory | 2048 MB (what I used in Lab 1) | **1024 MB** | 4 GB (Windows) + 2 GB = 6 GB does not fit into the 5.3 GB the host has free; Ubuntu Server uses ~200 MB at idle. |
| Ubuntu: OpenSSH server | not installed by the unattended install | **installed** (`sudo apt install -y openssh-server`) | So the developer can log in from a normal terminal and copy text out; without it SSH fails with `Connection aborted`. |
| Network adapter (both VMs) | NAT | **NAT Network `NatNetwork`** | The guests must see each other *and* the internet. |
| Windows: IPv4 address | automatic (DHCP) | **10.0.2.10/24, gateway 10.0.2.1, DNS 8.8.8.8** | DHCP gave Windows no gateway; a fixed address works and stays the same. |
| Windows Defender Firewall | blocks incoming ping | **rule "Allow ping in (ICMPv4)"** | The developer must be able to check that Linux reaches Windows; only ping is opened, the firewall stays on. |
| Windows: unattended install | offered by VirtualBox when the ISO is recognised | **off** | Aigerim sees and chooses the account and privacy settings herself. |
| Windows disk size | the wizard's suggestion | **64 GB** (typed in) | Windows 11's minimum; thin, so the file starts small. |
| Windows account | Microsoft account | **local account** | A test machine should not be tied to someone's personal Microsoft account. |
| Windows privacy settings | location, Find my device, offers: on; diagnostics: optional | **all off, diagnostics: Required only** | A test machine has no reason to send location or usage data. |
| Windows: Guest Additions | not installed | **installed** | The window resizes and copy-paste between laptop and guest works. |
| Shared clipboard | disabled | **bidirectional** | Needed to copy command output out of the Windows guest as text. |

Settings I kept at their default on purpose: 2 vCPU for each guest, EFI + Secure Boot + TPM 2.0 for Windows (VirtualBox turns these on when *Windows 11 (64-bit)* is chosen — BUILD.md tells Aigerim to check them), thin disks.

### Resources, calculated from my host's real numbers

Host (Lab 0, all from PowerShell): AMD Ryzen 7 7735HS, **8 cores / 16 threads**; **16 GB RAM** (one module), of which 707 MB is reserved by the hardware, so **15.3 GB usable**; disk C: **281.4 GB free**.

- **Memory.** With both VMs off and nothing open, Task Manager shows **10.0 GB in use and 5.3 GB available** (Lab 2, `labs/lab02-windows/evidence/host-memory-before.png`). So the guests can have at most about 5.3 GB together. Windows 11: 4 GB (its minimum, cannot go lower). That leaves 1.3 GB, so Ubuntu gets **1 GB**: 4 + 1 = **5 GB**, about **0.3 GB headroom** for the host. The headroom is small, so BUILD.md tells Aigerim to close other programs while both guests run, or to run one guest at a time. I tested what happens if the guests are given more than the host has (10 + 10 GB on 15.3 GB): available memory fell to 225 MB and committed memory hit its limit (Lab 2). It did not freeze at once, but there was nothing left in reserve.
- **CPU.** 2 vCPU + 2 vCPU = 4 of 16 threads, a quarter. The host keeps 12 threads. Never more than half of the host's cores per VM.
- **Disk.** Windows 64 GB + Ubuntu 25 GB = **89 GB in the worst case** if both disks fill up; both are thin, so they start much smaller. C: has 281.4 GB free, so even the worst case leaves about 190 GB. D: is a 7.1 GB driver partition with 1 GB free, not usable for VMs.

### Both guests reach the internet and each other

[evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt), unedited:

| From → to | Result |
|---|---|
| Ubuntu (10.0.2.4) → Windows (10.0.2.10) | 4 sent, 4 received, 0 % loss |
| Ubuntu → ubuntu.com | 4 sent, 4 received, 0 % loss |
| Windows → Ubuntu | 4 sent, 4 received, 0 % loss |
| Windows → ubuntu.com | 4 sent, 4 received, 0 % loss |

### One thing that went wrong, and how I fixed it

**Windows got no gateway from the NAT Network.** After moving `lab02-windows` from NAT to NAT Network, Windows could not reach anything: when I tried to remove its default route, PowerShell answered *"No matching MSFT_NetRoute objects found ... DestinationPrefix LIKE '0.0.0.0/0'"* — there was no default route at all ([screenshots/windows-static-ip-fix.png](screenshots/windows-static-ip-fix.png)). Ubuntu on the same NAT Network had received `10.0.2.4/24 ... dynamic` from DHCP, so the network itself was working and the problem was on the Windows side. I gave Windows a fixed address in the same network (10.0.2.10, gateway 10.0.2.1, DNS 8.8.8.8). After that Windows reached Ubuntu, 8.8.8.8 and ubuntu.com. This fix is Step 26 in BUILD.md, and the symptom is in its troubleshooting table.

Two more problems are written into BUILD.md's troubleshooting table:
- **Ubuntu → Windows ping timed out** (100 % loss) while Windows → Ubuntu worked. Cause: Windows Defender Firewall blocks incoming ICMP echo, and the guest's network profile is *Public*. Fix: the one ping rule above, not switching the firewall off ([screenshots/ubuntu-to-windows-timeout.png](screenshots/ubuntu-to-windows-timeout.png) → [screenshots/ubuntu-to-windows-after-rule.png](screenshots/ubuntu-to-windows-after-rule.png)).
- **Ubuntu pinged 8.8.8.8 but not ubuntu.com** (`Temporary failure in name resolution`). Pinging an IP worked, so only DNS was broken. After `sudo reboot` it worked with the same DNS server ([screenshots/ubuntu-dns-failure.png](screenshots/ubuntu-dns-failure.png) → [screenshots/ubuntu-dns-fixed-after-reboot.png](screenshots/ubuntu-dns-fixed-after-reboot.png)): the guest kept its old network state after the switch to NAT Network.

## 4. Evidence

- [evidence/01-new-vm-wizard.png](evidence/01-new-vm-wizard.png) — the VirtualBox new-VM wizard with my choices: name `lab02-windows`, the Windows 11 Enterprise evaluation ISO recognised as *Windows 11 Enterprise Evaluation (x64 / en-US)*, version Windows 11 (64-bit), unattended install off; below it 4096 MB memory, 2 CPU, EFI on
- [evidence/02-disk-settings.png](evidence/02-disk-settings.png) — the Windows disk: 64 GB VDI, *Pre-allocate full size* unticked, so it is thin (dynamically allocated)
- [evidence/03-network-adapter.png](evidence/03-network-adapter.png) — the network mode of a guest: Adapter 1 attached to *NAT Network*, name `NatNetwork` (the same network as Ubuntu, see `ip addr` 10.0.2.4/24 in Lab 2)
- [evidence/04-guest-uname.txt](evidence/04-guest-uname.txt) — `uname -a` and `lsb_release -a` inside the Linux guest, with the command lines: kernel 7.0.0-42-generic x86_64, Ubuntu 26.04.1 LTS
- [evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt) — Ubuntu → Windows, Ubuntu → internet, Windows → Ubuntu, Windows → internet; unedited output, all 0 % loss
- [evidence/06-snapshot-list.png](evidence/06-snapshot-list.png) — the snapshot list of `lab02-windows` with the clean snapshot `clean-install` (09.11.2026, VM powered off, memory back at 4096 MB) and *Current State* below it unchanged; above it an earlier snapshot from 08.11 named `скачив винд` is kept. Both VMs in the list show `(clean-install)` as their current snapshot

Step-by-step screenshots used by BUILD.md are in [screenshots/](screenshots/). The Ubuntu wizard screenshots (Steps 8–11 of BUILD.md) were retaken for this document with the name `test-ubuntu` and then cancelled, because `lab01-ubuntu` already exists on my laptop and the original Lab 1 wizard was not captured; the settings in them are the ones `lab01-ubuntu` was built with (1024 MB after Lab 2, 2 CPU, 25 GB thin disk, unattended install). The memory measurements behind the resource calculation are in `labs/lab02-windows/evidence/`.

## 5. What I would do differently

1. **Save the broken state before fixing it.** When Windows got no gateway I went straight to setting a fixed address and did not save `ipconfig` first, so the only record of the problem is the error from `Remove-NetRoute`. I also never found out *why* Windows' DHCP failed while Ubuntu's worked on the same network. Next time I would first run `ipconfig /all | Out-File before.txt`, then `ipconfig /release` and `ipconfig /renew`, and only use a fixed address if the renew fails.
2. **Pick the fixed address outside the DHCP range.** 10.0.2.10 may be inside the range that the NAT Network's DHCP hands out, so a third VM could one day get the same address. I would check the range in *File → Tools → Network → NAT Networks* and choose an address outside it.
3. **Take the Windows snapshot earlier.** I took `clean-install` only after the network and firewall changes; a snapshot straight after Guest Additions, and a second one after the network was working, would let the developer roll back to either point.
