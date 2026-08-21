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

*Troubleshooting Commands*
Used to confirm correct configuration paths:
.\splunk.exe btool inputs list --debug
.\splunk.exe btool outputs list --debug
These commands helped verif that the UF was reading the correct configuration files.
