Phase 2: Wazuh Installation
Date: 25 September 2026 Goal: Install Wazuh as an all-in-one deployment and access the web dashboard.

Wazuh Architecture
Component	Role
Wazuh indexer	Stores and searches alerts (based on OpenSearch)
Wazuh manager	Receives logs from agents and analyses them against detection rules
Filebeat	Ships alerts from the manager to the indexer
Wazuh dashboard	Web interface for investigating alerts
Wazuh agents	Run on monitored endpoints and forward their logs to the manager
An all-in-one install puts the indexer, manager and dashboard on one machine. That's fine for a lab; production environments spread them across multiple nodes for scale and resilience.

Installation
Run on the Wazuh-Server VM over SSH:

# Download the installation assistant
curl -so wazuh-install.sh https://packages.wazuh.com/4.14/wazuh-install.sh

# Verify it downloaded
ls -l wazuh-install.sh

# Run the all-in-one install (-a)
sudo bash ./wazuh-install.sh -a
Version 4.14 was chosen to match the Wazuh agent (4.14.8) that will be deployed later. Agents must not be newer than the manager.

What the installer did
Checked hardware requirements
Added the Wazuh package repository
Generated TLS certificates (root CA, admin, indexer, Filebeat, dashboard) so components talk to each other over encrypted connections
Installed and started the indexer, then initialised cluster security
Installed the manager and configured vulnerability detection
Installed Filebeat
Installed the dashboard and set up internal users
Total time: ~7.5 minutes.

<!-- Screenshot: install summary with the PASSWORD CROPPED OUT -->
Install summary

Accessing the Dashboard
Browsed to https://192.168.178.53 and logged in as admin.

The browser showed a "connection isn't private" warning because Wazuh uses a self-signed certificate that isn't signed by a trusted certificate authority. Acceptable for a lab server I built myself on my own network; on an unknown public site, this warning would be a red flag.

<!-- Screenshot: dashboard overview -->
Wazuh dashboard

First Observations
Even with no agents registered, the dashboard showed alerts in the first 24 hours:

Severity	Rule levels	Count
Critical	15+	0
High	12–14	0
Medium	7–11	124
Low	0–6	174
Why: The Wazuh manager monitors itself (agent 000). These alerts come from the server's own activity, e.g. package installs, sudo usage during setup, and Security Configuration Assessment (SCA) checks against the CIS benchmark.

Investigation notes
<!-- Fill these in after exploring Threat Hunting → Events and Configuration Assessment -->
Most common alert:
A sudo/PAM alert I found and what caused it:
A failed CIS check and why it matters:
Issues & Fixes
Issue 1: Install script printed to the screen instead of saving
What happened: The entire script source scrolled past, followed by ./wazuh-install.sh: No such file or directory. Cause: Typo in the curl flag. -sO (capital letter O = save using the remote filename) was mistyped, so curl wrote the file to stdout instead of disk. Fix: Used -so wazuh-install.sh to name the output file explicitly, and split download and install into separate commands so each step could be verified.

Issue 2: curl error in PowerShell
What happened: Invoke-WebRequest : A parameter cannot be found that matches parameter name 'so'. Cause: I ran the command in Windows PowerShell instead of on the Ubuntu VM. In Windows PowerShell 5.1, curl is an alias for Invoke-WebRequest, which has different parameters. Fix: SSH'd into the VM first, then confirmed the prompt showed hishaam@wazuh-server:~$ before running commands. Lesson: Always check the prompt to know which machine you're on. Running commands on the wrong host is a classic mistake in real incidents.

Credential Handling
The admin password was saved to a password manager immediately.
Screenshots were cropped so the password never appears.
It can be recovered from the install archive if needed:
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt | grep -A1 "'admin'"
wazuh-install-files.tar contains keys and passwords, so it must never be committed to Git.
What I Learned
SIEM architecture: collection → analysis → storage → visualisation
Why internal components use TLS and certificates
Self-signed vs CA-signed certificates
Wazuh rule severity levels
