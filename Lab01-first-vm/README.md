# Lab 1 - First Virtual Machine

## Resource justification
My host machine (Acer Nitro V15) has an Intel Core i5-13420H with 12 threads, 16 GB of RAM, and 90.6 GB of free disk space. For the Ubuntu Server VM, I allocated 2 vCPUs, which leaves 10 threads for the host OS and next week's Windows VM. I allocated 2 GB of RAM, which is more than enough for a console-only Linux guest, safely leaving 14 GB for the host and the future Windows VM. For storage, I assigned a 15 GB disk. Subtracting this from my 90.6 GB of free space leaves 75.6 GB, which is enough to accommodate next week's Windows guest installation.

## What you broke and how you confirmed it
I chose Option A. I deleted the hostname file using the command `sudo rm /etc/hostname` and rebooted the VM. I confirmed it was broken by running `cat /etc/hostname`, which returned "No such file or directory".

## What did not work
Initially, running the SSH command in PowerShell saved an empty 0-byte file because the connection was interrupted. I fixed this by ensuring the guest VM was running in a healthy state before executing the SSH command over port 22.
