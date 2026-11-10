# Build document: a safe test machine for a new developer

**For:** Aigerim T. — you do not need any IT background. Do the steps in order, one action at a time. Every step has a picture, a short **Why** (why this option and not another), and where it helps a **Check** (how you know the step worked).

**What you will have at the end:** two test computers that live inside your laptop as *virtual machines* (VMs):

| VM | System | What it is for |
|---|---|---|
| `lab01-ubuntu` | Ubuntu Server 26.04 LTS (Linux) | a Linux server, the kind most programs run on |
| `lab02-windows` | Windows 11 Enterprise (evaluation) | a Windows desktop, the kind most users have |

The developer can break anything inside them. The laptop is not affected, and you can bring a VM back to "freshly installed" in about one minute (Part F).

**Time:** about 2 hours, most of it waiting for installations. **Cost:** nothing. All software here is free and legal for an office.

> ⚠ **The Windows test machine stops working after 90 days** (it is a free evaluation). For a new developer, simply build a new one with this document. If a test machine must live longer than 90 days, the office needs a real Windows licence. Never type an office Windows key into a test machine and never download a "pre-activated" Windows from anywhere: that is pirated software.

Words used below: **host** = your real laptop. **guest** = a virtual machine. **ISO** = a file that contains an operating system installer, like a DVD in one file.

---

## Part 0 — Check the laptop first (5 minutes)

### Step 1. Check free memory
Close all programs. Press **Ctrl + Shift + Esc** → **Performance** → **Memory**. Read the number under **Available**.

![](screenshots/s00-host-memory.png)

**Why:** memory is the real limit. Windows itself already uses most of a 16 GB laptop (on mine: 10 GB in use, **5.3 GB available**). Both test machines together need **5 GB** (Windows 4 GB + Ubuntu 1 GB).
> **Check:** *Available* is **5.5 GB or more** → you can run both VMs at the same time. Less than that → you can still build both, but run only **one at a time**. Below 4.5 GB → ask IT for a laptop with more memory.

### Step 2. Check free disk space
Open **File Explorer → This PC** and look at **Local Disk (C:)**.

**Why:** the two VMs can grow up to 64 GB + 25 GB = 89 GB, plus about 10 GB for the two ISO files.
> **Check:** C: shows **at least 100 GB free**.

---

## Part A — Get the software (30 minutes, mostly downloading)

### Step 3. Download VirtualBox
Go to <https://www.virtualbox.org/wiki/Downloads> and click **Windows hosts**.

![](screenshots/s01-virtualbox-download.png)

**Why VirtualBox:** it is free also for companies (the base package is open source, GPLv3) and runs on any Windows laptop. VMware Workstation needs an account and a separate encryption password for Windows 11 that you could lose; Hyper-V is set up differently on every laptop.
⚠ Do **not** download the **Extension Pack** from the same page. It has a different licence that is **not free for companies**, and nothing here needs it.

### Step 4. Install VirtualBox
Open the downloaded file → **Next** on every screen → **Yes** when Windows asks for permission → **Finish**. If it warns that the network will be disconnected for a moment, click **Yes**.

![](screenshots/s02-virtualbox-about.png)

**Why the defaults:** they install exactly what is needed, nothing more.
> **Check:** open VirtualBox → **Help → About VirtualBox**. You see version **7.2** or newer.

### Step 5. Download Ubuntu Server
Go to <https://ubuntu.com/download/server> → **Download Ubuntu Server 26.04 LTS**. Save the ISO in **Downloads**.

![](screenshots/s03-ubuntu-download.png)

**Why Server and not Desktop:** Server has no graphical desktop and needs about 200 MB of memory when idle instead of 1–2 GB, which is what makes both VMs fit into the laptop. **LTS** means it gets security updates for 5 years.

### Step 6. Download Windows 11 Enterprise evaluation
Go to <https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise>, in the row **English (United States)** click **ISO – Enterprise download → 64-bit edition** (not the *LTSC* column; if a short form appears, fill it in). Save it in **Downloads**.

![](screenshots/s04-evalcenter.png)

**Why the evaluation:** it is the only legal way to get Windows for a test machine without paying and without a product key. The price is that it expires in 90 days (see the warning at the top).

---

## Part B — One network for both test machines (2 minutes)

### Step 7. Create the NAT Network
In VirtualBox: **File → Tools → Network** (*Файл → Инструменты → Сеть*, Ctrl+H) → tab **NAT Networks** (*Сети NAT*) → **Create** (*Создать*). A network called **NatNetwork** with IPv4 prefix **10.0.2.0/24** and **DHCP enabled** appears. Leave it as it is.

![](screenshots/s05-nat-network.png)

**Why NAT Network:** the developer needs two things at once — the two test machines must **see each other**, and both need **the internet**. NAT Network is the only mode that gives both:
- plain **NAT** (VirtualBox's default) gives each VM its own private router, so the two VMs can never see each other;
- **Host-only** lets them see each other but has **no internet**;
- **Bridged** puts the VMs on the office network as if they were separate computers: the campus/office network often blocks unknown devices, and it exposes test machines to everybody.
> **Check:** the list shows **NatNetwork**, **10.0.2.0/24**, DHCP server **Enabled**.

---

## Part C — The Linux test machine `lab01-ubuntu` (25 minutes)

### Step 8. Start the new-VM wizard
In VirtualBox click **Machine → New**. Fill in:
- **Name:** `lab01-ubuntu` (any name works; in my picture it says `test-ubuntu` because `lab01-ubuntu` already exists on my laptop)
- **ISO Image:** the Ubuntu Server file from Downloads
- **Proceed with Unattended Installation** (*Осуществить установку автоматически*): **ticked**

![](screenshots/u01-wizard-name-iso.png)

**Why unattended:** VirtualBox then answers all installer questions for you (disk layout, language, user). For a server that nobody looks at, this is the simplest and safest path.
> **Check:** *OS* became **Linux** and *Distribution* **Ubuntu** by itself. *Version* may show an older Ubuntu number (mine shows 25.04) — that is fine, VirtualBox just does not know the newest name yet; it must say **64-bit**. The red mark next to *Set up unattended guest OS installation* means the password is not set yet — that is the next step.

### Step 9. Set the user name and password
Open **Set up unattended guest OS installation**. Keep the user name **vboxuser**. Replace the password with your own and **write it on paper** (not in a file on the laptop). Hostname: `lab01-ubuntu` (my picture shows `test-ubuntu`, for the same reason as in Step 8).

![](screenshots/u02-wizard-unattended.png)

**Why:** the default password is the same on every VirtualBox installation, so everybody knows it.

### Step 10. Memory and processors
Open **Specify virtual hardware**: **Base Memory 1024 MB**, **Processors 2**.

![](screenshots/u03-wizard-hardware.png)

**Why 1024 MB:** see Step 1 — Windows needs 4 GB and the laptop has about 5 GB free, so Ubuntu gets 1 GB. A server without a desktop uses about 200 MB of it. Do not move the slider into the red part: there the host itself runs out of memory. **2 processors** are enough, and each VM should never get more than half of the laptop's processors.

### Step 11. The disk
Open **Specify virtual hard disk**: **Create a new virtual hard disk**, size **25 GB**, **Pre-allocate Full Size** **not** ticked. Click **Finish**.

![](screenshots/u04-wizard-disk.png)

**Why not pre-allocated:** the disk file starts small and grows only when the VM writes data, so 25 GB are not taken from the laptop on day one.
> **Check:** the installation starts by itself and runs for 10–15 minutes. Wait until you see `lab01-ubuntu login:`.

### Step 12. Move Ubuntu onto the shared network
Shut the VM down (log in → type `sudo poweroff` → Enter). Select `lab01-ubuntu` → **Settings → Network → Adapter 1** → **Attached to: NAT Network**, **Name: NatNetwork** → **OK**.

![](screenshots/u06-network-natnetwork.png)

**Why:** the wizard always attaches a new VM to plain NAT; only NAT Network lets it see the Windows VM (Step 7).

### Step 13. Log in and install the remote login (SSH)
Start the VM, log in as **vboxuser** with your password, and type, one line at a time, pressing Enter after each:
```bash
sudo apt update
sudo apt install -y openssh-server
```

![](screenshots/u07-openssh.png)

**Why:** the unattended installation does **not** include SSH. Without it the developer cannot log in from a normal terminal or copy text out of the server (`ssh` then fails with *Connection aborted*).
> **Check:** the last lines contain no `E:` errors. Running the second command again says *openssh-server is already the newest version*.

### Step 14. Check the network inside Ubuntu
Type:
```bash
ip -br a
ping -c 4 ubuntu.com
```

![](screenshots/u-ip-br-a-natnetwork.png)
![](screenshots/ubuntu-dns-fixed-after-reboot.png)

> **Check:** the line `enp0s3 UP` shows an address starting with **10.0.2.** (mine is **10.0.2.4**) — write it down, the Windows VM needs it. `ping` ends with **4 received, 0% packet loss**. If you see `Temporary failure in name resolution`, type `sudo reboot` and try again (see *When something goes wrong*).

---

## Part D — The Windows test machine `lab02-windows` (60 minutes)

### Step 15. Start the new-VM wizard
**Machine → New**:
- **Name:** `lab02-windows`
- **ISO Image:** the Windows 11 file from Downloads
- **Proceed with Unattended Installation:** **not** ticked

![](screenshots/w02-wizard-name-iso.png)

**Why no unattended install here:** you will choose the account and the privacy settings yourself (Steps 23–24) instead of VirtualBox choosing them silently.
> **Check:** *Edition* shows **Windows 11 Enterprise Evaluation** and *Version* switched to **Windows 11 (64-bit)**. If it still says Windows 10, as before an ISO is chosen (picture below), the ISO was not picked up — select it again.
>
> ![](screenshots/w01-wizard-before-iso.png)

### Step 16. Memory and processors
**Specify virtual hardware:** **Base Memory 4096 MB**, **Processors 2**, **Use EFI** ticked.

![](screenshots/w03-wizard-memory-cpu-efi.png)

**Why 4096 MB:** it is the minimum Windows 11 accepts — less and it refuses to install; more does not fit next to Ubuntu (Step 1).

### Step 17. The disk
**Specify virtual hard disk:** **Create a new virtual hard disk**, type **64** GB, **Pre-allocate Full Size** **not** ticked → **Finish**.

![](screenshots/w04-wizard-disk.png)

**Why 64 GB:** it is Windows 11's minimum; because the disk is not pre-allocated, the file starts small.

### Step 18. Check the security settings Windows 11 demands
Select `lab02-windows` → **Settings** → **System** (switch the window to **Expert** mode at the top if you don't see these): **TPM Version 2.0**, **Enable EFI** ✓, **Enable Secure Boot** ✓.

![](screenshots/w06-settings-system-tpm-uefi-secureboot.png)

**Why:** Windows 11 refuses to install without a TPM 2.0 chip, UEFI and Secure Boot. VirtualBox turns them on when *Windows 11* is selected, but if any one is missing you only find out at the end of the installer with *"This PC can't run Windows 11"*.

### Step 19. Put Windows on the same network
Still in **Settings → Network → Adapter 1**: **Attached to: NAT Network**, **Name: NatNetwork** → **OK**.

| Before (default) | After |
|---|---|
| ![](screenshots/w07-network-plain-nat-before.png) | ![](screenshots/w08-network-natnetwork.png) |

**Why:** the same reason as Step 12 — both VMs must be on **NatNetwork**, otherwise they cannot see each other.

### Step 20. Install Windows
Click **Start**. When the window says *"Press any key to boot from CD or DVD…"*, click inside the window and **press any key immediately**. Then: language → **Next** → **Install now** → **I don't have a product key** → **Windows 11 Enterprise Evaluation** → accept the licence → **Custom: Install Windows only** → **Drive 0 Unallocated Space** → **Next**.

![](screenshots/w09-installing.png)

**Why "I don't have a product key":** the evaluation works without one.
> **Check:** *Installing Windows 11* counts up to 100 %. The VM restarts several times — **do not** press a key at the "boot from CD" message again, or the installation starts over.

### Step 21. Create a local account
When setup asks for a Microsoft account, choose **Sign-in options → Domain join instead** (if that option is missing, use any Microsoft account created for the office). You reach **Who's going to use this device?** Type a name → **Next**, then a password and three security questions. **Write the password on paper.**

![](screenshots/w10-local-account.png)

**Why a local account:** a test machine should not be tied to anyone's personal Microsoft account, and it must keep working when the developer leaves.

### Step 22. Switch off every privacy option
On **Choose privacy settings for your device**, scroll and switch everything to **No** (diagnostic data: **Required only**) → **Accept**.

| Default: on | Switched off |
|---|---|
| ![](screenshots/w11-privacy-default-on.png) | ![](screenshots/w12-privacy-location-off.png) |
| ![](screenshots/w13-privacy-find-device-off.png) | ![](screenshots/w14-privacy-diagnostics-required.png) |
| ![](screenshots/w15-privacy-offers-off.png) | |

**Why:** a test machine has no reason to send its location or usage data to Microsoft.

### Step 23. Wait for the first updates and log in
Setup downloads updates (several minutes), then shows the login screen. Log in with your password.

| | |
|---|---|
| ![](screenshots/w16-oobe-update.png) | ![](screenshots/w17-login.png) |

![](screenshots/w18-desktop-enterprise-eval.png)

> **Check:** bottom right of the desktop: **Windows 11 Enterprise Evaluation — Windows License valid for 90 days**. Write down today's date + 90 days.

### Step 24. Install Guest Additions
In the VM window menu: **Devices → Insert Guest Additions CD image**. In Windows open **File Explorer → CD Drive (D:) VirtualBox Guest Additions** → double-click **VBoxWindowsAdditions** → **Yes** → **Next** on every screen → **Install** → **Reboot now**.

| | | |
|---|---|---|
| ![](screenshots/w19-guest-additions-cd.png) | ![](screenshots/w20-guest-additions-exe.png) | ![](screenshots/w21-guest-additions-installing.png) |

**Why:** without them the Windows screen stays tiny and copy-paste between the laptop and the VM does not work.
> **Check:** after the reboot the desktop fills the VM window when you resize it.

### Step 25. Turn on copy-paste
VM window menu: **Devices → Shared Clipboard → Bidirectional**.

![](screenshots/w25-shared-clipboard.png)

**Why:** it is off by default. The developer needs it to copy commands in and results out.

### Step 26. Give Windows its network address
Click **Start**, type `powershell`, choose **Run as administrator** → **Yes**.

![](screenshots/w22-powershell-admin.png)

Copy these four lines into the blue window (right-click pastes), pressing Enter after each:
```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled
Remove-NetIPAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.2.10 -PrefixLength 24 -DefaultGateway 10.0.2.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8
```

![](screenshots/windows-static-ip-fix.png)

**Why:** on my laptop Windows did not receive a working address from the NAT Network by itself (it had no *gateway*, the way out to the internet). These lines give it a fixed address on the same network as Ubuntu: **10.0.2.10**, way out through **10.0.2.1**, names looked up at **8.8.8.8**. A fixed address also means the developer always knows where Windows is. Red text after the second line (*No matching ... objects found*) is harmless — it only means there was nothing to remove.
> **Check:** type `ipconfig`. You see **IPv4 Address 10.0.2.10**, **Default Gateway 10.0.2.1**.

### Step 27. Allow ping — and only ping — through the Windows firewall
In the same administrator PowerShell window:
```powershell
New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
Get-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" | Format-List DisplayName,Enabled,Direction,Action,Profile
```

![](screenshots/firewall-rule-check.png)

**Why:** *ping* is the standard way to check that two computers see each other. Windows' firewall blocks incoming ping by default, because a new network is treated as **Public** (the strictest setting). This rule opens exactly ping and nothing else. **Never switch the firewall off** — that makes ping work but leaves the whole machine open.
> **Check:** the output says **Enabled: True, Direction: Inbound, Action: Allow**.

---

## Part E — Check that everything is connected (5 minutes)

Both VMs must be running.

### Step 28. From Windows: to Ubuntu and to the internet
In PowerShell in Windows (use the Ubuntu address from Step 14):
```powershell
ping 10.0.2.4
ping ubuntu.com
```

![](screenshots/windows-ipconfig-ping-ubuntu.png)

> **Check:** both end with **Received = 4, Lost = 0 (0% loss)**.

### Step 29. From Ubuntu: to Windows and to the internet
In the same PowerShell window (this logs in to Ubuntu over SSH and runs the pings there; type `yes` the first time, then the Ubuntu password):
```powershell
ssh vboxuser@10.0.2.4 "ping -c 4 10.0.2.10; ping -c 4 ubuntu.com"
```

![](screenshots/ubuntu-to-windows-after-rule.png)

> **Check:** both end with **4 received, 0% packet loss**. My full, unedited output is in [evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt).

---

## Part F — The clean snapshot, and how to roll back

This is the part that makes the test machines safe: a **snapshot** is a saved copy of the whole VM at one moment. Whatever the developer breaks later, you return to this moment.

### Step 30. Take the clean snapshot of each VM
Shut both VMs down (Ubuntu: `sudo poweroff`; Windows: **Start → Power → Shut down**). Select the VM → the **Snapshots** tab (camera icon, top right) → **Take** → Name: `clean-install`, Description: what is inside, e.g. *fresh install + SSH* / *Windows 11 eval + Guest Additions + network + ping rule* → **OK**. Do this for both VMs.

![](evidence/06-snapshot-list.png)
![](screenshots/u-snapshots-ubuntu.png)

**Why with the VM switched off:** the snapshot then contains only the disk and settings, it is smaller, and restoring it gives a cleanly started machine.
> **Check:** both VMs show **(clean-install)** after their name in the list on the left.

### Step 31. Roll back after the developer breaks something
Shut the VM down completely — the list must say **Powered Off**, not *Saved* or *Running*. **Snapshots** tab → click **clean-install** → **Restore** → in the question, **untick** *Create a snapshot of the current machine state* (unless you want to keep the broken state) → **Restore** → start the VM.

![](screenshots/f02-restore-dialog.png)

**Why fully powered off:** on my first attempt the VM was not fully off, the restore saved the broken state as a new snapshot instead of going back, and nothing was fixed.
> **Check:** the VM name in the list shows **(clean-install)** again, and the VM starts exactly as it was after installation.

---

## When something goes wrong

| You see | Cause | What to do |
|---|---|---|
| *"This PC can't run Windows 11"* | TPM 2.0, EFI or Secure Boot is off in the VM | Shut the VM down, Step 18, start the installation again |
| *"Press any key to boot from CD"* missed; black screen or *No bootable medium* | The key was not pressed in time | **Machine → Reset**, press a key immediately |
| Setup insists on a Microsoft account | Normal Windows 11 setup screen | **Sign-in options → Domain join instead** (Step 21) |
| Windows screen is tiny, copy-paste does nothing | No Guest Additions, or clipboard off | Steps 24 and 25 |
| `ipconfig` in Windows shows **no Default Gateway**, or an address starting with **169.254** | Windows got no address from the NAT Network | Step 26 |
| Ubuntu → Windows ping: **100% packet loss**, but Windows → Ubuntu works | Windows firewall blocks incoming ping | Step 27 — never switch the firewall off |
| The two VMs cannot ping each other at all | One of them is on plain NAT, or on a different network | Steps 12 and 19: both **NAT Network → NatNetwork** |
| Ubuntu: `ping 8.8.8.8` works but `ping ubuntu.com` says **Temporary failure in name resolution** | Ubuntu kept old network settings after its network was changed | `sudo reboot`, then try again |
| Ubuntu still shows an address that does not start with 10.0.2. | The address is given at start-up | `sudo reboot` |
| `ssh` says **Connection aborted** or **Connection refused** | SSH server not installed in Ubuntu | Step 13 |
| VirtualBox refuses to start a VM, or the laptop becomes very slow | The VMs were given more memory than the laptop has free | Shut one VM down; check Step 1 and keep 4096 + 1024 MB |
| A small green turtle in the VM window's status bar | Windows' own Hyper-V is running and VirtualBox runs on top of it | Nothing to fix — it is only slower. Do not change Windows security settings to remove it |
| After **Restore** the VM is still broken | The VM was not fully powered off | Step 31: shut down until it says **Powered Off**, then restore again |
| Windows shows *"Your Windows license will expire soon"* | The 90-day evaluation is ending | Build a fresh `lab02-windows` with Part D, or buy a licence |
