A home Security Operations Centre (SOC) lab built to practise deploying a SIEM, collecting logs from endpoints, and detecting and investigating security events.

Status: In progress. Phases 1–2 complete (server built, Wazuh deployed). Next: Windows endpoint + agent.

Goal
Build a working SIEM environment from scratch on my own hardware, connect monitored endpoints, generate real security events, and write custom detection rules. The lab mirrors, on a small scale, how a real SOC collects and analyses logs.

Lab Architecture
flowchart LR
    subgraph Host["Host PC: Windows 11 (32GB RAM) + VirtualBox 7.2"]
        W["Wazuh-Server VM<br/>Ubuntu 24.04.5 LTS<br/>192.168.178.53<br/>8GB RAM / 4 vCPU / 50GB"]
        V["Win11-Victim VM<br/>Windows 11 Enterprise<br/>(planned)"]
    end
    V -- "Wazuh agent<br/>logs & events (1514/tcp)" --> W
    A["Analyst browser"] -- "HTTPS dashboard (443/tcp)" --> W
Component	Details
Hypervisor	Oracle VirtualBox 7.2.20
SIEM server	Ubuntu Server 24.04.5 LTS, Wazuh 4.14.8 (all-in-one)
Networking	Bridged adapter: VMs sit on the home LAN (192.168.178.0/24)
Endpoint (planned)	Windows 11 Enterprise evaluation + Wazuh agent 4.14.8
Tools & Technologies
Wazuh 4.14.8: open-source SIEM/XDR (indexer, manager, dashboard, Filebeat)
Ubuntu Server 24.04 LTS: Linux server administration
VirtualBox: virtualisation and virtual networking
Windows 11: monitored endpoint (planned)
Skills Demonstrated
Virtual machine provisioning and resource planning
Linux server installation, disk partitioning (LVM), and patching
Remote administration over SSH
SIEM deployment and architecture (indexer / manager / dashboard)
Initial alert triage and log analysis
Troubleshooting and root-cause analysis
Progress
Phase	Description	Status
01	Environment setup: VirtualBox + Ubuntu Server VM	✅ Done
02	Wazuh all-in-one installation + dashboard access	✅ Done
03	Windows 11 endpoint + Wazuh agent deployment	⏳ Next
04	Generate security events and investigate alerts	⬜ Planned
05	Write custom detection rules	⬜ Planned
Key Lessons Learned (so far)
Check the defaults. Ubuntu's installer only allocated half the disk to the root filesystem by default, which a log-heavy SIEM would fill quickly.
Check which machine you're on before running commands. I ran a Linux command in Windows PowerShell by mistake; the shell prompt tells you where you are.
Version compatibility matters. Wazuh agents must not be newer than the manager, so I matched both at 4.14.x. I also chose Ubuntu 24.04 over the newer 26.04 because it is an officially supported OS.
See lab-notes.md for raw working notes.

Security Note
No credentials, keys, or the wazuh-install-files.tar archive are published in this repository. IP addresses shown are private (RFC 1918) lab addresses.
