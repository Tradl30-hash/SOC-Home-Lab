**1. Windows 10 VM Setup**
The endpoint was built as a dedicated victim machine inside VMware Workstation, configured with:
- Memory: 8 GB
- Processors: 4
- Hard Disk (NVMe): 60 GB
- Networking: NAT + Host‑Only
- Installation Media: Windows 10 Pro ISO

This VM provides a clean environment for authentication testing, Sysmon telemetry, and brute‑force simulation.
No domain was used — all testing was performed with local accounts only.

**2. Splunk Universal Forwarder Installation**
The Splunk Universal Forwarder (UF) was installed to forward Windows logs to the Splunk Enterprise server.
*Serviice Verification:*
- Get-Service SplunkForwarder

*Returned:*
- Status: Running
- Name: SplunkForwarder
- DisplayName: SplunkForwarder

*Forwarding Configuration*
Set the Splunk server as the forwarding destination:
- "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" add forward-server 192.168.138.129:9997

*Verification Commands*
On Splunk Server: 
- sudo /opt/splunk/bin/splunk list forward-server

On Windows UF: 
- cd "C:\Program Files\SplunkUniversalForwarder\bin"
- .\splunk.exe list forward-server
- Get-Service SplunkForwarder

Enable Auto-Start:
- .\splunk.exe enable boot-start

Installed Splunk_TA_Windows on enpoint machine through splunk website.
- C:\Program Files\SplunkUniversalForwarder\etc\apps\Splunk_TA_Windows


**3. Inputs and Outputs Configuration**
Files in this repo
- SOC-Home-Lab/Windows-Endpoint/Inputs.conf — UF monitor definitions and XML rendering settings.
- SOC-Home-Lab/Windows-Endpoint/Outputs.conf — UF tcpout stanza pointing to the Splunk indexer.

What these files do

- inputs.conf tells the Universal Forwarder which Windows event channels to monitor and ensures events are forwarded in XML so Splunk can extract fields. In this lab the monitored channels are: Security; System; Application; Microsoft-Windows-PowerShell/Operational; Microsoft-Windows-Sysmon/Operational. The file also enables XML rendering (renderXml = 1) for the Security channel so fields like TargetUserName, LogonType, and Status are available to Splunk_TA_windows.

outputs.conf configures the forward destination for the UF. It contains the tcpout stanza that points to your Splunk server IP and port (192.168.138.129:9997).

Where the UF reads these files on the Windows host

*Troubleshooting Commands*
Used to confirm correct configuration paths:
- .\splunk.exe btool inputs list --debug
- .\splunk.exe btool outputs list --debug
These commands helped verif that the UF was reading the correct configuration files.

**4. Local Account Creation (Victim / Attacker / Admin)**
*Using lusrmgr.msc, three local accounts were created to support authentication and brute‑force testing:*
- Victim — target account for failed logon attempts
- Attacker — account used to generate brute‑force activity
- Admin — administrative account for configuration and management

Each account was assigned a password and verified through successful login.
These accounts generate the key Windows Security Events:

4625 — Failed logon

4624 — Successful logon

These events are essential for brute‑force correlation detections.

**5. Windows Event Log Forwarding (XML Rendering Enabled)**
To ensure proper field extraction (e.g., TargetUserName, EventID, Status), Windows Event Logs were configured to forward in XML format.

inputs.conf includes:
- renderXml = 1
This setting is required for Splunk_TA_windows to parse Windows Event Logs correctly.

**6. Sysmon Installation & Configuration**
*Sysmon was installed using Sysinternals to provide detailed endpoint telemetry, including:*
- Process creation
- Network connections
- Registry modifications
- File creation events

*Sysmon Configuration Issue (Troubleshooting)*
The downloaded sysmon-config.html file would not convert to XML even after enabling file extensions.
Fix:

Right‑click file → Properties → “Opens with” → Change → Notepad

Only then did the file properly convert to .xml.

*Installation Command*
- sysmon.exe -i Sysmon-Config.xml
The Sysmon configuration used in this lab is stored in:

Windows-Endpoint/Sysmon-Config.xml

Sysmon Log Verification
Sysmon logs were verified through Windows Event Viewer:

Applications and Services Logs → Microsoft → Windows → Sysmon → Operational

Sysmon telemetry is forwarded through the UF and will be used for future detections beyond brute‑force authentication.
