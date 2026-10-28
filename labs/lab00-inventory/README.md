# Lab 0 — Inventory your own machine
Pair: <<ЗАПОЛНИТЬ: фамилия>> — individual work    Driver: <<фамилия>> (whole session)
Machine: Acer Nitro ANV15-41, Windows 11 Pro (10.0.26200)
Date: <<ЗАПОЛНИТЬ: дата>>

## What I did
Every value below came from PowerShell; the graphical "About" screen was not used.
Each evidence file contains the command (`PS> ...`) followed by its raw output.

```powershell
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed | Format-List
Get-CimInstance Win32_PhysicalMemory | Select-Object DeviceLocator, Manufacturer, PartNumber, @{n='GB';e={$_.Capacity/1GB}}, Speed, ConfiguredClockSpeed | Format-Table -AutoSize
Get-CimInstance Win32_PhysicalMemoryArray | Select-Object MemoryDevices, @{n='MaxGB';e={$_.MaxCapacity/1MB}} | Format-List
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, BusType, @{n='SizeGB';e={[math]::Round($_.Size/1GB,1)}} | Format-Table -AutoSize
Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter, FileSystemLabel, FileSystem, @{n='SizeGB';e={[math]::Round($_.Size/1GB,1)}}, @{n='FreeGB';e={[math]::Round($_.SizeRemaining/1GB,1)}} | Format-Table -AutoSize
$env:firmware_type
Get-CimInstance Win32_BIOS | Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate | Format-List
Get-CimInstance Win32_Processor | Select-Object VirtualizationFirmwareEnabled, VMMonitorModeExtensions, SecondLevelAddressTranslationExtensions | Format-List
Get-CimInstance Win32_ComputerSystem | Select-Object HypervisorPresent | Format-List
```

## Result
| What | Value | Where I got it |
|---|---|---|
| CPU model | AMD Ryzen 7 7735HS with Radeon Graphics | `Get-CimInstance Win32_Processor` → `Name` |
| CPU cores | 8 | `Get-CimInstance Win32_Processor` → `NumberOfCores` |
| CPU threads | 16 | `Get-CimInstance Win32_Processor` → `NumberOfLogicalProcessors` |
| Total memory | 16 GB | `Get-CimInstance Win32_PhysicalMemory` → `Capacity` |
| Memory modules | **1** × 16 GB, Kingston ACR56S46BS8HA-16KM, in slot DIMM 0 | `Get-CimInstance Win32_PhysicalMemory` → `DeviceLocator`, `PartNumber`, `Capacity` |
| Memory speed | module rated 5600 MT/s, **running at 4800 MT/s** | `Get-CimInstance Win32_PhysicalMemory` → `Speed`, `ConfiguredClockSpeed` |
| Memory slots | 2 (one used, one free), firmware maximum 64 GB | `Get-CimInstance Win32_PhysicalMemoryArray` → `MemoryDevices`, `MaxCapacity` |
| Disk model | HFS001TEJ9X110N, 953.9 GB | `Get-PhysicalDisk` → `FriendlyName`, `Size` |
| Disk type | NVMe SSD | `Get-PhysicalDisk` → `MediaType`, `BusType` |
| Free space on the VM volume (C:) | **281.4 GB** free of 944.8 GB | `Get-Volume` → `SizeRemaining`, `Size` |
| Other volume (D:) | `WINDRIVER`, 7.1 GB, 1 GB free | `Get-Volume` → `FileSystemLabel`, `Size`, `SizeRemaining` |
| Firmware type | UEFI | `$env:firmware_type` |
| Firmware version | INSYDE Corp., V1.52 | `Get-CimInstance Win32_BIOS` → `Manufacturer`, `SMBIOSBIOSVersion` |
| Firmware date | 2026-08-04 | `Get-CimInstance Win32_BIOS` → `ReleaseDate` |
| Hardware virtualization enabled | **Yes**: `VirtualizationFirmwareEnabled` = True | `Get-CimInstance Win32_Processor` |
| Hypervisor already running | **Yes**: `HypervisorPresent` = True | `Get-CimInstance Win32_ComputerSystem` |

**Memory and upgrades.** 16 GB is a single module in slot DIMM 0, and the system reports two memory slots.
So one slot is free: the laptop can be upgraded by **adding** a second module, without throwing the first away.
A matching second module would also switch the memory from single-channel to dual-channel.

**Where VMs go.** On C:, which has 281.4 GB free. D: is not a separate disk: it is a 7.1 GB `WINDRIVER` partition
(Acer's driver partition) with 1 GB free, unusable for VMs.

## What did not work the first time
- **The virtualization flags looked like "no virtualization".** `VMMonitorModeExtensions` and
  `SecondLevelAddressTranslationExtensions` came back **False**, which reads as if the CPU had no VT/AMD-V and no SLAT.
  But `VirtualizationFirmwareEnabled` is True and `HypervisorPresent` is True: Windows 11 is already running its own
  hypervisor (Hyper-V / Virtualization-based security). When a hypervisor is running, Windows itself is a guest
  and these flags are not reported to it. So False here does not mean "missing". Virtualization is on and in use.
  Consequence for Lab 1: VirtualBox runs on top of Hyper-V, not directly on the hardware.
- **Memory runs slower than the module's rating.** The module is rated 5600 MT/s, but `ConfiguredClockSpeed` is 4800.
  The memory runs at the speed the platform sets, not at the speed printed on the module.
- <<ЕСЛИ БЫЛО: что-то, что у неё реально пошло не так при выполнении — иначе удали эту строку>>

## Evidence
- [evidence/cpu.txt](evidence/cpu.txt) — CPU model, 8 cores, 16 threads
- [evidence/memory.txt](evidence/memory.txt) — one 16 GB module, rated 5600 and running at 4800, two slots
- [evidence/disk.txt](evidence/disk.txt) — NVMe SSD model and size; free space on C: and D:
- [evidence/firmware.txt](evidence/firmware.txt) — UEFI, INSYDE V1.52
- [evidence/virtualization.txt](evidence/virtualization.txt) — virtualization enabled, hypervisor running
