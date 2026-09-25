Phase 1: Environment Setup
Date: 24–25 September 2026 Goal: Create an Ubuntu Server VM to host the Wazuh SIEM.

1. Hypervisor
Installed Oracle VirtualBox 7.2.20 on a Windows 11 host (32GB RAM). Chose Expert Mode in VirtualBox so all network and hardware options are visible.

2. Choosing the OS
I first downloaded Ubuntu 26.04, but switched to Ubuntu Server 24.04.5 LTS.

Why: Wazuh's installer checks the OS against a list of supported versions. 24.04 is officially supported; 26.04 was too new and might fail the check or cause hard-to-diagnose issues. For infrastructure, a stable, supported LTS release is a better choice than the newest one.

3. VM Configuration
Setting	Value	Reason
Name	Wazuh-Server	
RAM	8192 MB	Wazuh indexer (OpenSearch-based) is memory-hungry; 8GB is the recommended size
vCPUs	4	Indexing and rule analysis are CPU-intensive
Disk	50 GB	Room for stored logs and alerts
Unattended install	Skipped	To choose install options manually
Network	Bridged Adapter (Wi-Fi)	Gives the VM its own IP on the home LAN so other VMs and the host can reach it
<!-- Screenshot: VM summary screen -->
VM summary

4. Ubuntu Installation
Network: Received IP 192.168.178.53/24 via DHCP, which confirms bridged networking worked (same subnet as the host).
Storage: See the issue below. Expanded root filesystem to use the full disk.
Profile: Hostname wazuh-server.
SSH: Installed OpenSSH server for remote administration.
Snaps: None selected (minimal install = smaller attack surface).
<!-- Screenshot: storage config after fixing -->
Storage configuration

5. Post-install
Updated the system:

sudo apt update && sudo apt upgrade -y
sudo reboot
Then connected over SSH from the Windows host so commands could be pasted rather than typed:

ssh hishaam@192.168.178.53
<!-- Screenshot: first login showing IP address -->
First login

Issues & Fixes
Issue 1: Accidentally opened "Create bond"
What happened: Selected the "Create bond" option on the network screen. Fix: Cancelled. Bonding combines multiple network interfaces, which isn't needed with a single adapter.

Issue 2: Root filesystem only used half the disk
What happened: Ubuntu's default LVM layout allocated only ~24GB to /, leaving ~24GB unused in the volume group. Why it matters: A SIEM continuously stores logs; 24GB could fill up and crash the indexer. Fix: Edited the ubuntu-lv logical volume in the installer and set it to the maximum size (~48GB). Lesson: Don't blindly accept installer defaults on servers. Check them against what the workload needs.

Issue 3: Boot appeared to hang at "Loading essential drivers"
What happened: Boot seemed stuck on kernel messages (raid6: avx2x4 gen()). Fix: Waited; it was the kernel's RAID speed benchmark, which runs slower inside a VM.

What I Learned
How bridged vs NAT networking affects VM reachability
LVM basics: volume groups vs logical volumes, and how the Ubuntu installer uses them
Resource planning for a SIEM workload
Shutting down a VM safely (sudo shutdown now / save state) vs "power off" (like pulling the plug)
