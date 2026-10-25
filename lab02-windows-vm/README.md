# Lab 02 — A Windows guest, and a network between them

## 1. Network Configuration
* Network Mode: NAT (VMware Workstation Virtual Network vmnet8).
* Why NAT was chosen:
  * Both VMs can access the external internet seamlessly through the host's network connection without requiring separate physical IP addresses.
  * Unlike Bridged mode, NAT is immune to campus/enterprise MAC address filtering and 802.1X security policies that drop unauthorized MAC addresses.
  * NAT isolates guest machines into their own private subnet (192.168.44.0/24), enabling reliable local communication between guests via a virtual switch.

## 2. Network Topology & Addresses
* Host Machine: Windows 11 Laptop (16 GB RAM)
* Linux Guest (Ubuntu 24.04/26.04 LTS): 192.168.44.129 (interface ens33)
* Windows Guest (Windows 11 Enterprise): 192.168.44.130 (interface Ethernet0)
* Default Gateway / NAT Device: 192.168.44.2

### Network Diagram
              [ Physical Host / Internet ]
                           |
                     (NAT Router)
                     192.168.44.2
                           |
       +-------------------+-------------------+
       |                                       |
 [ Ubuntu Guest ]                      [ Windows 11 Guest ]
 192.168.44.129                        192.168.44.130
 (ens33)                               (Ethernet0)

## 3. Connectivity & Firewall Resolution
* Initial Observation: Pinging from Windows to Linux succeeded immediately. However, pinging from Linux to Windows resulted in 100% packet loss.
* Root Cause: Windows Defender Firewall default inbound policies block unsolicited ICMPv4 Echo Request packets on public/private profiles.
* Proper Remediation: Rather than disabling the Windows Firewall completely, a specific scoped inbound firewall rule was added via administrative PowerShell:
  New-NetFirewallRule -DisplayName "Lab02 - allow ping (ICMPv4 echo)" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -RemoteAddress LocalSubnet -Action Allow
* After applying the rule, bidirectional ICMP connectivity was fully confirmed with 0% packet loss.

## 4. Memory Overcommitment Experiment
* Host Physical Memory: 16.0 GB DDR4.
* Baseline Measurement: Normal RAM usage hovered around ~8.8 GB (56%) with VMs running standard allocations (Ubuntu: 2 GB, Windows: 4 GB).
* Overcommitment Configuration: Memory allocation for both VMs was raised to 10 GB each, totalling 20 GB allocated memory (exceeding physical host capacity by 4 GB).
* Observed Host Behavior:
  * Both virtual machines were able to start, but the host physical memory usage rapidly escalated to 99% (15.6 GB / 15.7 GB consumed, <100 MB free).
  * Committed memory rose significantly (from 13.1 GB to 23.2 GB out of the 27.4 GB pagefile pool).
  * The Windows host aggressively utilized memory compression and disk paging (swap) on the NVMe SSD to accommodate the guest ballooning and virtual RAM allocations, leading to heavy storage I/O and general host latency.
* Post-experiment: VM allocations were safely reset back to 2 GB (Ubuntu) and 4 GB (Windows).

## 5. Artifacts and Evidence
All required evidence files are located in the evidence/ directory:
* host-memory-before.png: Baseline host RAM usage prior to overcommitment.
* host-memory-after.png: Host RAM usage peaking at 99% under overcommitted load.
* ipconfig-windows.txt: Full IP configuration dump from the Windows guest.
* ip-addr-linux.txt: Network interface and address assignment from the Ubuntu guest.
* ping-both-ways.txt: Successful bidirectional ping verification logs between guests.
