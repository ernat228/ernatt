# Lab 0 - Inventory

|What|Value|Where I got it|
|-|-|-|
|CPU model|13th Gen Intel(R) Core(TM) i5-13420H|Get-CimInstance Win32\_Processor|
|Cores / threads|8 / 12|Get-CimInstance Win32\_Processor|
|RAM total|16 GB|Get-CimInstance Win32\_PhysicalMemory (sum of Capacity)|
|RAM modules / speed|1 1 module(s), 3200 MHz|Get-CimInstance Win32\_PhysicalMemory|
|Disk model / type|KINGSTON OM8PGP4512Q-AA, SSD, NVMe|Get-PhysicalDisk|
|Free space on VM volume (C:)|90.6 GB|Get-Volume -DriveLetter C|
|Firmware type / version / date|Uefi (Get-ComputerInfo BiosFirmwareType), V1.30, 06/13/2025 05:00:00|Get-CimInstance Win32\_BIOS, Confirm-SecureBootUEFI|
|Hardware virtualization enabled|False|Get-CimInstance Win32\_Processor (VirtualizationFirmwareEnabled)|

## Virtualization

VirtualizationFirmwareEnabled = False, but Win32_ComputerSystem.HypervisorPresent = True, so virtualization is enabled and used by Windows (read via Get-CimInstance Win32\_Processor).

## What did not work

My first attempt failed: I mistyped a command (vmkdir) and ran it from C:\\Windows\\System32, so the evidence folder was not created. A try/catch with Out-File in one line also gave a parser error. I fixed the commands and re-ran everything from my Documents folder.
The graphical About screen was not used.



