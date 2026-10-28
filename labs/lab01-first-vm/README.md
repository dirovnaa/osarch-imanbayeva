# Lab 1 — Your first virtual machine
Pair: <<Imanbayeva Daiana>> — individual work, as instructed in class for this lab
Machine: Acer Nitro ANV15-41, Windows 11 Pro (10.0.26200); guest: Ubuntu Server 26.04.1 LTS (`lab01-ubuntu`) in VirtualBox

## What I did
1. Created VM `lab01-ubuntu` in VirtualBox: 2 vCPU, 2048 MB RAM, 25 GB dynamically allocated VDI,
   NAT network with port forwarding host 2222 → guest 22.
2. Ubuntu Server 26.04.1 LTS was installed by VirtualBox's unattended install. It did not include
   OpenSSH server, so I added it before the snapshot:
```bash
sudo apt install -y openssh-server
```
3. Saved the guest release from the host over SSH, so the evidence is text, not a picture:
```powershell
ssh -p 2222 vboxuser@127.0.0.1 "echo '$ lsb_release -a'; lsb_release -a" | Out-File -Encoding utf8 guest-release.txt
```
4. Powered the guest off and took the snapshot before changing anything:
```powershell
VBoxManage snapshot "lab01-ubuntu" take clean-install --description "fresh install + openssh-server, nothing else changed"
```
5. Broke the system on purpose and checked it after a reboot:
```bash
cat /etc/hostname
sudo rm /etc/hostname
sudo reboot
# after reboot
cat /etc/hostname
hostnamectl
```
6. Powered off, restored `clean-install` (the broken state was kept as snapshot `broken`), booted and checked again:
```bash
cat /etc/hostname
hostnamectl
```

## Resource allocation
Host numbers, all from PowerShell:

```powershell
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors   # Ryzen 7 7735HS, 8 / 16
Get-CimInstance Win32_PhysicalMemory | Select-Object Manufacturer, @{n='GB';e={$_.Capacity/1GB}}, Speed   # 1 x 16 GB Kingston, 5600
Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter, @{n='SizeGB';e={[math]::Round($_.Size/1GB,1)}}, @{n='FreeGB';e={[math]::Round($_.SizeRemaining/1GB,1)}}   # C: 944.8 / 281.5 free, D: 7.1 / 1 free
```
Idle memory use was taken from Task Manager → Performance → Memory with VirtualBox closed
(see [evidence/host-memory.png](evidence/host-memory.png)): **9.9 GB in use of 15.3 GB usable**
(16 GB installed, about 0.7 GB reserved by the hardware for the integrated GPU).

Host (see [Lab 0](../lab00-inventory/README.md)): AMD Ryzen 7 7735HS (8 cores / 16 threads),
16 GB RAM as **one** 16 GB module running at 4800 MT/s, C: with 281.4 GB free. Hyper-V is running on the host,
so VirtualBox runs on top of it.

I gave the guest 2 vCPU of 16 threads, 2 GB of 16 GB RAM and a 25 GB dynamically allocated disk.

- **Memory is the real limit, not the 16 GB on the label.** With nothing open, Windows already uses 9.9 GB,
  so about 5.4 GB is actually free. 2 GB is enough for Ubuntu Server without a graphical shell and leaves
  the host about 3.4 GB. In Lab 2 a 4 GB Windows guest on top of this would need 6 GB in total, more than
  is free, so before Lab 2 I have to find what holds the memory or lower the Linux guest to 1–1.5 GB.
- **CPU:** 2 vCPU are one eighth of the host's 16 threads, so the host and a second guest keep plenty.
- **Disk:** the VM lives on C:, which has 281.4 GB free. The 25 GB disk is dynamic, so it only takes the space
  actually written. D: is only 7.1 GB with 1 GB free, so it is not usable for VMs.

## Result
| | Broken | After rollback |
|---|---|---|
| `cat /etc/hostname` | No such file or directory | `lab01-ubuntu` |
| Static hostname | (unset) | `lab01-ubuntu` |
| Transient hostname | `lab01-ubuntu` | not shown, same as static |

The prompt stayed `vboxuser@lab01-ubuntu` even when broken: a transient hostname was set at boot, so the damage
was invisible in the prompt and only `hostnamectl` showed that the static hostname was gone.

## What did not work the first time
- **The host's memory was almost full before the VM started.** The first measurement showed about 12–13 GB
  used of 16, with only 3.2 GB available, and I first suspected a virus. Closing VirtualBox brought it down to
  9.9 GB. The processes in Task Manager add up to only about 2.5 GB, so most of the used memory is not held
  by any visible program; it belongs to Windows itself, drivers and the hypervisor (Hyper-V is running,
  see Lab 0), plus about 0.7 GB reserved for the integrated GPU. Nothing suspicious appeared in the
  process list. The VM size was chosen from the 9.9 GB idle number, not from the 16 GB total.
- **SSH to the guest failed: `kex_exchange_identification: read: Connection aborted`.** The port forward
  2222 → 22 was set, so VirtualBox accepted the connection, but nothing was listening on port 22 inside the guest.
  Ubuntu had been installed by VirtualBox's unattended install (user `vboxuser`, the whole 25 GB disk as `/`),
  and that install does not include OpenSSH server. `systemctl status ssh` then said `inactive (dead)`, which
  looked like another failure, but Ubuntu starts SSH through `ssh.socket` on the first connection.
  Fixed with `sudo apt install -y openssh-server` **before** taking the snapshot, so `clean-install` includes SSH.
- **The first rollback did not roll anything back.** After the first restore attempt the window title said
  `lab01-ubuntu (broken)` and `/etc/hostname` was still missing: the current state had been saved as a snapshot
  `broken`, but the machine had not gone back to `clean-install`. The second attempt, with the VM fully
  powered off (status "Powered Off" in the manager) and `clean-install` selected, worked: the title changed
  to `(clean-install)` and the hostname came back.

## Evidence
- [evidence/vm-settings.png](evidence/vm-settings.png) — CPU, RAM and disk as configured
- [evidence/host-memory.png](evidence/host-memory.png) — host memory in use with VirtualBox closed
- [evidence/guest-release.txt](evidence/guest-release.txt) — the guest runs Ubuntu Server LTS, with the command that produced it
- [evidence/broken.png](evidence/broken.png) — the guest after deleting /etc/hostname and rebooting
- [evidence/restored.png](evidence/restored.png) — after the rollback /etc/hostname is back
- [evidence/snapshot-list.png](evidence/snapshot-list.png) — the snapshots after the rollback
