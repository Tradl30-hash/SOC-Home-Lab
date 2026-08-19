1. Windows 10 VM Setup
The endpoint was built as a dedicated victim machine inside VMware Workstation, configured with:
- Memory 8 GB
- Processors 4
- Hard Disk(NVMe) 60 GB
- NAT + Host‑Only networking
- Standard Windows 10 Pro installation ISO file

Clean environment for authentication testing and Sysmon telemetry

No domain — local accounts only

This machine acts as the attacker/victim environment for generating real authentication and Sysmon events.

2. Splunk Universal Forwarder Installation
The Splunk Universal Forwarder (UF) was installed on the Windows 10 endpoint to forward logs to the Splunk server.

Forwarding configuration:
Get-Service SplunkForwarder
Returned: Status - running | Name - SplunkForwarder | DisplayName - SplunkForwarder
Set the forward address for logs to begin flowing.
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" add forward-server 192.168.138.129:9997

Verification:
- sudo /opt/splunk/bin/splunk list forward-server
- PS C:\Windows\System32> cd "C\Program Files\SplunkuniversalForwarder\bin" ./Splunk.exe list forward-server
- Get-Service SplunkForwarder

enabled auto start for splunk UF:
PS C:\Program Files\SplunkUniversalForwarder\bin> .\Splunk.exe enable boot-start

used commands such as: to check if files were coming through the correct local file.
./splunk.exe btool inputs list --debug 
./splunk.exe btool outputs list --debug 


Local Account Creation (Victim / Attacker / Admin)
Using lusrmgr.msc, I created three local accounts to support brute‑force and authentication testing:

- Victim - target account for failed logon attempts
- Attacker - account used to generate brute‑force activity
- Admin - administrative account for configuration and management

Each account was assigned a password and verified through successful login.
These accounts are essential for producing EventID 4625 (failed logon) and EventID 4624 (successful logon).

3. Splunk Universal Forwarder Installation
The Splunk Universal Forwarder (UF) was installed on the Windows 10 endpoint to forward logs to the Splunk server.

Forwarding configuration:

Code
splunk add forward-server <splunk-server-ip>:9997
splunk add monitor WinEventLog://Security
splunk add monitor WinEventLog://System
splunk add monitor WinEventLog://Application
Verification:

Code
splunk list forward-server
splunk list monitor
This ensures Windows Event Logs are continuously forwarded to Splunk.

3. Verified Sysmon logs were running through Windows Event Viewer:
   Applications and Services > Microsoft > Windows > Sysmon > Operational

5. Sysmon Installation & Configuration Installed both sysinternals SYSMON and sysmon-config

Sysmon was installed using Sysinternals to provide detailed endpoint telemetry such as:
Process creation
Network connections
Registry modifications
File creation events

Troubleshooted: Sysmon-config.html
After file was downloaded, it would'nt open up as a XML file no matter what I saved it as. Even after enabling (View <- File name extensions) so that the ending file extension wasn't hidden. 
Fix: open files menu(Right-click) < properties < change.. open with < Notepad) Only then will the file be changed to an xml extension.

Installation command:

Code
sysmon.exe -i Sysmon-Config.xml
The Sysmon configuration used in this lab is stored in:

Windows-Endpoint/Sysmon-Config.xml

Sysmon logs are forwarded through the UF and used for future detections beyond brute‑force authentication.
Talk about Windows event viewer to check sysmon logs.
