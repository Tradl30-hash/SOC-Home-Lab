Brute Force Detection – Splunk Correlated Alert (Windows Security Logs)
A hands‑on detection engineering project demonstrating how to identify brute‑force attacking attempts by correlating Windows Security EventID Logs (4625 failed logons + 4624 successful logons) ingested into Splunk.

  Overview
This detection identifies when a user(Victim) account experiences 5+ failed logon attempts followed by a successful logon within a short time window.
The workflow includes:

Forwarding Windows Security logs (XML) via Splunk Universal Forwarder.

Parsing XML fields using Splunk Add‑on for Microsoft Windows (Splunk_TA_windows).

Extracting key fields such as TargetUserName, EventID, and Source.

Writing SPL to correlate failed and successful authentication events.

Producing a clean result table showing brute‑force activity.

This project demonstrates practical SIEM detection engineering skills applicable to SOC Analyst roles.

📁 Lab Architecture
  **Windows 10 Endpoint**
Generates Security Event Logs (4624 / 4625)

Runs Splunk Universal Forwarder

Sends XML logs to Splunk

  **Ubuntu Splunk Server**
Receives and indexes logs

Runs Splunk Enterprise

Hosts Splunk_TA_windows for XML field extraction

Executes detection SPL

🔧 Required Add‑on
Splunk Add‑on for Microsoft Windows (Splunk_TA_windows)
This add‑on enables XML parsing and field extraction for Windows Event Logs.
**Deployment Note: This add-on must be installed on **both** the windows endpoint to collect t he logs and the Ubuntu Splunk server to properly parse and index the fields.**

Key extracted fields used in this detection:

TargetUserName

source

EventID

LogonType

Index=main

Without this add‑on, XML logs appear as raw text and correlation is not possible.

🔍 Detection Logic
The detection works by:

Counting failed logon attempts (EventCode 4625) per username

Filtering users with 5 or more failures

Joining those results with successful logons (EventCode 4624)

Producing a final table showing brute‑force activity

**SPL Query (Correlated Detection)**
index=main source="XmlWinEventLog:Security" EventID=4625
| stats count as failed_attempts by TargetUserName
| where failed_attempts >= 5
| join TargetUserName [
    search index=main source="XmlWinEventLog:Security" EventID=4624
    | stats count as success by TargetUserName
]
| table TargetUserName failed_attempts success

| TargetUserName | failed_attempts | success |
|     Victim     |        8        |     1   |
