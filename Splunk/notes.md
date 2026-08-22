**Splunk Server Setup Notes (Ubuntu 22.04.5)**
   *Environment*
Hypervisor: VMware Workstation
ISO: ubuntu-22.04.5-live-server-amd64.iso
VM Specs:
- Memory: 12.1 GB
- CPU: 4 cores
- Disk: 100 GB
- Network Adapter 1: NAT
- Network Adapter 2: Host‑Only
- Splunk Admin User: splunkadmin

**Installing Splunk Enterprise**
Started Splunk for the first time:
- sudo -u splunk /opt/splunk/bin/splunk start --accept-license
This initialized Splunk Web and prompted for admin credentials.

**Enabling Auto‑Start on Boot**
Splunk requires a dedicated service user for boot‑start. Root cannot be used.
Created a service user:
- sudo adduser splunkd
Assigned ownership of splunk directories:
- sudo chown -R splunkd:splunk /opt/splunk
Enabled boot-start:
- sudo /opt/splunk/bin/splunk enable boot-start -user splunkd
Restarted Splunk: 
- sudo systemctl restart splunk
- sudo -u splunk /opt/splunk/bin/splunk status
Splunk now auto‑starts on boot.

**Accessing Splunk Web**
Opened Splunk Web from the host machine:
- http://192.168.138.129:8000
**Enabled log receiving (Port 9997)**
Configured Splunk to receive logs from the Universal Forwarder.
- Settings → Forwarding and Receiving → Configure Receiving → Add Port 9997
Verified:
- sudo ss -tlnp | grep 9997
Output:
- LISTEN 0 128 0.0.0.0:9997

**VirtualBox Issues**
Originally attempted this setup in Oracle VirtualBox.
Spent 2–3 days troubleshooting ingestion issues — Windows logs were coming through except Sysmon, which refused to ingest properly.
Completed more research on Oracle VirtualBox and found it's unreliable and commonly has broken host only netowrk adapters and performance issues.

Switched to VMware Workstation and ingestion immediately became stable and reliable.

**Log Ingestion Verification**
Confirmed logs were flowing:
- index=*
verified forwarder connection:
- sudo -u splunk /opt/splunk/bin/splunk list forward-server

**XML Field Extraction Issue (Solved)**
Initially, XML logs were ingesting but fields were not extracted.
Queries like:
- index=* sourcetype=WinEventLog:Security EventID=4625
returned events, but fields such as TargetUserName were missing.
*Root cause*:  Splunk_TA_windows was installed on the Windows UF, but not on the Splunk server.
*Fix*:  Installed Splunk_TA_windows on Splunk Web:
- Apps → Manage Apps → Install app from file
After installation, XML fields were extracted correctly and SPL detections began working.

This was the moment the entire pipeline came together.
