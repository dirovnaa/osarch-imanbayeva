# Lab 4 — Partition, mirror, fail, recover

Pair: Imanbayeva / Yermukhan    Driver first half: Imanbayeva
Machine: Acer Nitro ANV15-41, Windows 11 Pro (10.0.26200); guests `lab01-ubuntu` (Ubuntu Server 26.04.1 LTS, 1 GB RAM) and `lab02-windows` (Windows 11 Enterprise evaluation) in VirtualBox 7.2
Date: 12.11.2026

## What I did

### 0. Extra disks
VirtualBox → *Settings → Storage → Controller: SATA → Add hard disk → Create*, VDI, dynamically allocated, 2 GB each:

- `lab01-ubuntu`: three new disks `lab01-ubuntu_1.vdi`, `_2.vdi`, `_3.vdi` → inside the guest `sdb`, `sdc`, `sdd` ([screenshots/ubuntu-storage-4-disks.png](screenshots/ubuntu-storage-4-disks.png))
- `lab02-windows`: one new disk `lab02-windows_1.vdi` → inside the guest *Disk 1* ([screenshots/windows-storage-extra-disk.png](screenshots/windows-storage-extra-disk.png))

All Linux commands were run over SSH from the host (`ssh -p 2222 vboxuser@127.0.0.1`, a port-forwarding rule on the NAT Network → 10.0.2.4:22, [screenshots/natnetwork-port-forward-ssh.png](screenshots/natnetwork-port-forward-ssh.png)). Each evidence file was written with a small helper that prints the command before its output, `run() { echo "\$ $*"; "$@" 2>&1; echo; }`, piped into `tee`, then copied to the host with `scp -P 2222 "vboxuser@127.0.0.1:lab04/*.txt" .` — the files are unedited.

### 1. Windows: partition, format, letter
PowerShell as Administrator in `lab02-windows`:
```powershell
$env:firmware_type
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter
Get-Partition -DiskNumber 1 | Where-Object DriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel LAB04
Get-Disk; Get-Partition -DiskNumber 1
```

**MBR or GPT, and why — GPT.** The firmware is **UEFI** (`$env:firmware_type`), and the system disk (Disk 0) is already GPT: a UEFI machine boots Windows 11 only from a GPT disk. For the new data disk I chose GPT too: it is the default on UEFI, it keeps a backup copy of the partition table at the end of the disk with checksums (MBR has one copy, and if it is damaged the partitions are lost), it allows 128 partitions instead of 4 primary, and it works above 2 TB. MBR would only be needed for a disk that must boot an old BIOS-only machine or be read by a very old OS — neither applies here.

### 2. Linux: partition two disks and build the mirror
```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS                                   # → lsblk-before.txt
sudo apt install -y mdadm
for d in sdb sdc; do sudo parted -s /dev/$d mklabel gpt mkpart raid 1MiB 100% set 1 raid on; done
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 --metadata=1.2 --bitmap=internal --run /dev/sdb1 /dev/sdc1
sudo mkfs.ext4 -L mirror /dev/md0
sudo mkdir -p /mnt/mirror && sudo mount /dev/md0 /mnt/mirror
sudo mdadm --wait /dev/md0                                                   # wait for the initial sync
cat /proc/mdstat; sudo mdadm --detail /dev/md0                               # → array-healthy.txt
```
Both disks got a GPT table with one partition flagged `raid`; the mirror is built from the partitions (`sdb1`, `sdc1`), not from the raw disks, so every disk carries a label that says what it belongs to.

### 3. A recognisable file and its checksum
```bash
echo "Lab 4 - Imanbayeva Daiana - written $(date)" | sudo tee /mnt/mirror/proof.txt
sudo dd if=/dev/urandom of=/mnt/mirror/random-100M.bin bs=1M count=100
sync
sha256sum /mnt/mirror/proof.txt /mnt/mirror/random-100M.bin                  # → checksum-before.txt
```
One small text file you can read, and one 100 MB file of random data, so that a single changed bit anywhere would change the checksum.

### 4. Fail one disk
```bash
sudo mdadm /dev/md0 --fail /dev/sdc1
sudo mdadm /dev/md0 --remove /dev/sdc1
cat /proc/mdstat; sudo mdadm --detail /dev/md0                               # → array-degraded.txt
sudo sysctl vm.drop_caches=3                                                 # forget cached file contents
sha256sum /mnt/mirror/proof.txt /mnt/mirror/random-100M.bin                  # → checksum-after-failure.txt
```
The page cache was dropped before the second checksum, so the files were read again from the one disk that was left, not from memory.

### 5. Add the third disk and rebuild
```bash
sudo parted -s /dev/sdd mklabel gpt mkpart raid 1MiB 100% set 1 raid on
sudo mdadm /dev/md0 --add /dev/sdd1
cat /proc/mdstat; sleep 3; cat /proc/mdstat                                  # watch the recovery
sudo mdadm --wait /dev/md0
cat /proc/mdstat; sudo mdadm --detail /dev/md0
sha256sum /mnt/mirror/proof.txt /mnt/mirror/random-100M.bin                  # → array-rebuilt.txt
```

## Result

### Layout before and after — Linux guest

Before ([evidence/lsblk-before.txt](evidence/lsblk-before.txt)):
```
sda      25G disk
├─sda1    1M part
└─sda2   25G part ext4     /
sdb       2G disk
sdc       2G disk
sdd       2G disk
```
After the rebuild (from [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) and [evidence/array-healthy.txt](evidence/array-healthy.txt)):
```
sda      25G disk                       (system disk, not touched)
sdb       2G disk  GPT
└─sdb1    2G part  linux_raid_member
  └─md0   2G raid1 ext4  /mnt/mirror
sdc       2G disk  GPT   sdc1 — failed and removed from md0
sdd       2G disk  GPT
└─sdd1    2G part  linux_raid_member
  └─md0   2G raid1 ext4  /mnt/mirror
```

### Layout before and after — Windows guest ([evidence/windows-partition.txt](evidence/windows-partition.txt))

| | Disk 0 (system, 64 GB) | Disk 1 (new, 2 GB) |
|---|---|---|
| Before | GPT | RAW (no partition table) |
| After | GPT, unchanged | **GPT**: partition 1 *Reserved* (MSR, 16 MB, no letter) + partition 2 *Basic*, **E:**, NTFS, label `LAB04`, 2.13 GB |

Windows created the 16 MB *Microsoft Reserved* partition by itself, which is why my partition is number 2 and slightly smaller than the disk.

### The three states of the array

| State | `/proc/mdstat` | `mdadm --detail` | Members | Evidence |
|---|---|---|---|---|
| Healthy | `[2/2] [UU]` | `State : clean`, 2 active, 0 failed | `sdb1`, `sdc1` active sync | [array-healthy.txt](evidence/array-healthy.txt) |
| Degraded | `[2/1] [U_]` | `State : clean, degraded`, 1 active | `sdb1` active sync, slot 1 *removed* | [array-degraded.txt](evidence/array-degraded.txt) |
| Rebuilding | `[2/1] [U_]`, `recovery = 0.3%` → `26.4%` at 184 MB/s | — | `sdd1[2]` being written | [array-rebuilt.txt](evidence/array-rebuilt.txt) |
| Rebuilt | `[2/2] [UU]` | `State : clean`, 2 active, 0 failed | `sdb1`, **`sdd1`** active sync | [array-rebuilt.txt](evidence/array-rebuilt.txt) |

### Checksums — the data survived

| File | Before the failure | With one disk left | After the rebuild |
|---|---|---|---|
| `proof.txt` | `20ae90ce…80375830` | `20ae90ce…80375830` | `20ae90ce…80375830` |
| `random-100M.bin` | `be478332…0aa26f7640` | `be478332…0aa26f7640` | `be478332…0aa26f7640` |

All three SHA-256 sums are identical in full (see the files), so the filesystem stayed readable and not one bit changed when half of the mirror was lost.

### Your colleague deleted the wrong file from the mirror an hour ago. Does RAID 1 help? Why?

No. RAID 1 writes every change to both disks at the same moment, so the deletion was copied to the second disk within milliseconds and the file is gone from both halves. A mirror protects against a disk **dying**, not against a person or a program doing the wrong thing — for that you need a **backup** or a snapshot taken before the deletion, kept somewhere the mistake could not reach.

## What did not work the first time

1. **The first mirror was not created at all.** `mdadm 4.5` first asked *"enable write-intent bitmap? [y/N]"*; my `y` was answered with `mdadm: bad option`, the next question *"Continue creating array? [y/N]"* got no answer and mdadm printed `assuming no` and `create aborted` ([screenshots/mdadm-create-aborted.png](screenshots/mdadm-create-aborted.png)). The disks were untouched. I gave both answers as options instead of typing them: `--bitmap=internal` and `--run` (and `--metadata=1.2` to make the default explicit). Then: `mdadm: array /dev/md0 started.`
2. **VirtualBox refused to create the first disk:** *"The given path 'lab04-disk1.vdi' is not fully qualified"* (`VBOX_E_FILE_ERROR`, [screenshots/disk-create-error-path.png](screenshots/disk-create-error-path.png)). I had typed only a name, and VirtualBox needs a full path for a disk created from the *Storage* dialog. Keeping the suggested name (with its folder) worked; that is why the disks are called `lab01-ubuntu_1.vdi` and not `lab04-disk1.vdi`.
3. **I almost replaced the Ubuntu system disk.** While adding disks I had a disk selected instead of the controller, and the *add* button replaced the selected disk: the list showed `_1`, `_2`, `_3` and **no `lab01-ubuntu.vdi`** ([screenshots/mistake-system-disk-replaced.png](screenshots/mistake-system-disk-replaced.png)). I saw it before pressing OK, chose *Cancel → Discard changes*, and attached the three existing disks again, each time selecting **Controller: SATA** first. The system disk stayed on port 0.
4. **Two evidence outputs were cut short.** (a) In `array-degraded.txt` the lines under `--fail` and `--remove` are empty: mdadm writes those messages (`hot removed /dev/sdc1 from /dev/md0`) to stderr, and `tee` only captured stdout. The message is in [screenshots/array-degraded-1.png](screenshots/array-degraded-1.png); after that I changed the helper to `"$@" 2>&1`, so the later files include stderr. (b) Copying `Get-Partition` from Windows through the clipboard stopped after the first row every time: the MSR partition has no drive letter, PowerShell shows it as a NUL character, and the clipboard treats NUL as the end of the text. Printing `-` instead of the empty letter fixed it.

## Evidence

- [evidence/lsblk-before.txt](evidence/lsblk-before.txt) — the Linux guest before anything: system disk `sda` and three empty 2 GB disks `sdb`, `sdc`, `sdd`
- [evidence/array-healthy.txt](evidence/array-healthy.txt) — GPT partition with the `raid` flag on `sdb`; `md0` RAID 1 `[2/2] [UU]`, `State : clean`, `sdb1` + `sdc1` active sync; ext4 mounted on `/mnt/mirror`
- [evidence/checksum-before.txt](evidence/checksum-before.txt) — the two files on the mirror and their SHA-256 before the failure
- [evidence/array-degraded.txt](evidence/array-degraded.txt) — after failing and removing `sdc1`: `[2/1] [U_]`, `State : clean, degraded`, slot 1 *removed*
- [evidence/checksum-after-failure.txt](evidence/checksum-after-failure.txt) — with only `sdb1` left and the cache dropped: same files, same SHA-256
- [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) — `sdd1` added, recovery at 0.3 % and 26.4 %, then `[2/2] [UU]`, `State : clean`, `sdb1` + `sdd1`; SHA-256 still the same
- [evidence/windows-partition.txt](evidence/windows-partition.txt) — Windows: UEFI firmware, Disk 1 RAW → GPT, partition E: NTFS `LAB04`, the automatic MSR partition
- [screenshots/](screenshots/) — the same steps as screenshots, plus the VirtualBox storage settings and the errors described above
