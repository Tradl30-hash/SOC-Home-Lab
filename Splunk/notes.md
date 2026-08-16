installed ISO file on host computer, created a new VM on VMware used Ubuntu-22.04.5-=liver-server-amd64.iso to get a live splunk server.
memory=12.1gb | processors=4 | hard disck=100GB | network adapter=NAT | network adapter2=Host-Only 
Prompts allowed me to create my Admin: Splunkadmin

Started splunk for the first time with command
sudo -u splunk /opt/splunk/bin/splunk start --accept-license

After the license was accepted and the splunk server was up and running the next command was to get splunk to auto start on boot.
This gave me some trouble as I had to create a specific account to give permissions of splunk to for auto boot since root couldn't do it.
sudo adduser splunkd
sudo [password] splunkd
sudo chown -R splunkd:splunk /opt/splunk
sudo /opt/splunk/bin/splunk enable boot-start -splunkd splunk
sudo systemctl restart splunk
sudo -u splunk /opt/splunk/bin/splunk status

Server is now up and running with auto start enabled. It's time to setup the splunk web server.
http://192.168.138.129:8000

Enabled receiving on port 9997 for UF logs.
settings > Forwarding and receiving > configure receiving > add port 9997
OR on the Ubuntu command line
sudo -u splunk /opt/splunk/bin/splunk enable listen 9997

**I first started this journey on Oracle VirtualBox I spent 2-3 days altering and configuring the windows and splunk servers and settings. Only to find out that Oracle VirtualBox was very unreliable, I switched over to VMware which was a much smoother operation. Oracle VirtualBox was ingesting all the logs except for Sysmon and I couldn't troulbeshoot it to work for Sysmon until I swapped over to VMware.**

At this time logs were being ingested and Splunk was on auto-start on boot. index=* was showing plenty of leg ingestion.
sudo -u splunk /opt/splunk/bin/splunk list forward-server

sudo ss -tlnp | grep 9997
Listen 0  128   0.0.0.0:9997

At this point I stopped configuring splunk for a long time and focused my energy on the Windows10 machine. Come to find out I am getting all of my XML logs ingestion but when querying for such logs I wasn't able to extract the fields I wanted. IE 'index=* sourcetype=WinEventLog:Security EventID 4625' this was able to find logs yes, but my SPL wasn't working and was unable to find TargetUserName. It wasn't until later I found out I needed to also install Splunk_TA_Windows addon ONTO the actual splunk web server. So I downloaded the addon onto my main computer host.
Launch splunk web server > under apps "Manage" > install app from file > installed splunk addon
Now I have all the extracted fields I need to correctly query XML logs. This was the moment everything was finally tied together.
