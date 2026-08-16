installed ISO file on host computer, created a new VM on VMware used Ubuntu-22.04.5-=liver-server-amd64.iso to get a live splunk server.

Prompts allowed me to create my User: Splunkadmin

Started splunk for the first time with command
sudo -u splunk /opt/splunk/bin/splunk start --accept-license

After the license was accepted and the splunk server was up and running the next command was to get splunk to auto-start on boot.
sudo /opt/splunk/bin/splunk enable boot-start -Splunkadmin splunk
sudo systemctl restart splunk

