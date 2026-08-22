1. Windows 10 VM Setup
The Windows 10 endpoint was built as a dedicated victim machine inside VMware Workstation, configured with:

Memory: 8 GB

Processors: 4

Hard Disk (NVMe): 60 GB

Networking: NAT + Host‑Only

Installation Media: Windows 10 Pro ISO

This VM provides a clean environment for authentication testing, Sysmon telemetry, and brute‑force simulation.
All testing was performed using local accounts only — no domain.

2. Splunk Universal Forwarder Installation
The Splunk Universal Forwarder (UF) was installed to forward Windows logs to the Splunk Enterprise server.

Service Verification
Code
Get-Service SplunkForwarder
Returned:

Status: Running

Name: SplunkForwarder

DisplayName: SplunkForwarder

Forwarding Configuration
Set the Splunk server as the forwarding destination:

Code
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" add forward-server 192.168.138.129:9997
Verification Commands
On Splunk Server:

Code
sudo /opt/splunk/bin/splunk list forward-server
On Windows UF:

Code
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe list forward-server
Get-Service SplunkForwarder
Enable Auto‑Start
Code
.\splunk.exe enable boot-start
Splunk_TA_Windows Installation (Endpoint)
Installed the Splunk_TA_Windows add-on on the Windows endpoint:

Code
C:\Program Files\SplunkUniversalForwarder\etc\apps\Splunk_TA_Windows
This ensures proper sourcetyping and tagging before logs reach Splunk.

3. Inputs and Outputs Configuration
Files in this repository
SOC-Home-Lab/Windows-Endpoint/Inputs.conf — UF monitor definitions and XML rendering settings

SOC-Home-Lab/Windows-Endpoint/Outputs.conf — UF tcpout stanza pointing to the Splunk indexer

Purpose of these files
inputs.conf  
Defines which Windows event channels are monitored:

Security

System

Application

Microsoft-Windows-PowerShell/Operational

Microsoft-Windows-Sysmon/Operational

Also enables XML rendering:

Code
renderXml = 1
This ensures fields like TargetUserName, LogonType, and Status are extracted correctly by Splunk_TA_windows.

outputs.conf  
Defines the forward destination:

Code
server = 192.168.138.129:9997
Where UF reads these files
Code
C:\Program Files\SplunkUniversalForwarder\etc\system\local\
Troubleshooting Commands
Used to confirm correct configuration paths:

Code
.\splunk.exe btool inputs list --debug
.\splunk.exe btool outputs list --debug
These commands verify that the UF is reading the correct configuration files.

4. Local Account Creation (Victim / Attacker / Admin)
Using lusrmgr.msc, three local accounts were created to support authentication and brute‑force testing:

Victim — target account for failed logon attempts

Attacker — account used to generate brute‑force activity

Admin — administrative account for configuration and management

Each account was assigned a password and verified through successful login.

These accounts generate the key Windows Security Events:

4625 — Failed logon

4624 — Successful logon

These events are essential for brute‑force correlation detections.

5. Windows Event Log Forwarding (XML Rendering Enabled)
To ensure proper field extraction (e.g., TargetUserName, EventID, Status), Windows Event Logs were configured to forward in XML format.

inputs.conf includes:

Code
renderXml = 1
This setting is required for Splunk_TA_windows to parse Windows Event Logs correctly.

6. Sysmon Installation & Configuration
Sysmon was installed using Sysinternals to provide detailed endpoint telemetry, including:

Process creation

Network connections

Registry modifications

File creation events

Sysmon Configuration Issue (Troubleshooting)
The downloaded sysmon-config.html file would not convert to XML even after enabling file extensions.

Fix:

Right‑click file → Properties → “Opens with” → Change → Notepad

Only then did the file properly convert to .xml.

Installation Command
Code
sysmon.exe -i Sysmon-Config.xml
Sysmon configuration stored in:

Code
Windows-Endpoint/Sysmon-Config.xml
Sysmon Log Verification
Verified through Windows Event Viewer:

Code
Applications and Services Logs → Microsoft → Windows → Sysmon → Operational
Sysmon telemetry is forwarded through the UF and used for future detections beyond brute‑force authentication.

7. Splunk Server Setup (Ubuntu 22.04.5)
Environment
Hypervisor: VMware Workstation

ISO: ubuntu-22.04.5-live-server-amd64.iso

Memory: 12.1 GB

CPU: 4 cores

Disk: 100 GB

Networking: NAT + Host‑Only

Splunk Admin User: splunkadmin

Installing Splunk Enterprise
Code
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
Enabling Auto‑Start
Code
sudo adduser splunkd
sudo chown -R splunkd:splunk /opt/splunk
sudo /opt/splunk/bin/splunk enable boot-start -user splunkd
sudo systemctl restart splunk
sudo -u splunk /opt/splunk/bin/splunk status
Accessing Splunk Web
Code
http://192.168.138.129:8000
Enable Receiving on Port 9997
Splunk Web → Settings → Forwarding and Receiving → Add Port 9997

Verification:

Code
sudo ss -tlnp | grep 9997
Output:

Code
LISTEN 0 128 0.0.0.0:9997
8. VirtualBox Issues (Resolved by Switching to VMware)
Originally attempted this setup in Oracle VirtualBox:

Windows logs ingested

Sysmon logs did not ingest

Host‑only networking was unstable

Known VirtualBox issues surfaced (broken adapters, performance problems)

Switched to VMware Workstation, and ingestion became stable immediately.

9. Log Ingestion Verification
Confirmed logs were flowing:

Code
index=*
Verified forwarder connection:

Code
sudo -u splunk /opt/splunk/bin/splunk list forward-server
10. XML Field Extraction Issue (Solved)
XML logs were ingesting but fields were missing:

Code
index=* sourcetype=WinEventLog:Security EventID=4625
Events appeared, but fields like TargetUserName were missing.

Root Cause
Splunk_TA_windows was installed on the Windows UF, but not on the Splunk server.

Fix
Installed Splunk_TA_windows on Splunk Web:

Apps → Manage Apps → Install app from file

After installation:

XML fields extracted correctly

SPL detections worked

Brute‑force correlation logic succeeded

This completed the ingestion pipeline.
