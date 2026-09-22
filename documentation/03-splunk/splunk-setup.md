             My Splunk Setup and Connectivity
Overview

This section documents the Splunk infrastructure used in the my SOC Lab.

The lab uses Splunk Enterprise as the central SIEM platform and a Splunk Universal Forwarder on Kali Linux to forward security logs to the Splunk server.

Lab Architecture
Kali Linux
IP: 172.20.10.5
        |
        | Splunk Universal Forwarder
        | TCP 9997
        v
Splunk Enterprise
IP: 172.20.10.2
1. Splunk Enterprise Status

Splunk Enterprise was verified from the macOS terminal using:

/Applications/Splunk/bin/splunk status

The command confirmed that splunkd and its helper processes were running successfully.

Evidence

[(My Splunk Status)](https://github.com/HIDDDS/MuhideenShield-SOC-Lab/blob/cf8b636d058d95363f3fe25c720f3a1550658ba7/screenshots/01-kali-network/Splunk-status.png)

2. Splunk Receiving Port

The Splunk configuration was inspected using:

/Applications/Splunk/bin/splunk btool inputs list --debug | grep -A 10 '\[splunktcp'

The configuration contained:

[splunktcp://9997]

This confirms that a Splunk TCP receiving input is configured on port 9997.

Evidence
![Splunk TCP 9997 Receiver](https://github.com/HIDDDS/MuhideenShield-SOC-Lab/blob/90dcec9a1f79be4f331704e96d189c054a856c14/screenshots/01-kali-network/splunk-receiver-9997.png)





3. Kali-to-Splunk Network Connectivity

Connectivity from the Kali Linux endpoint to the Splunk server was tested using Netcat:

"nc -vz 172.20.10.2 9997"

The test returned:

"[172.20.10.2] 9997 (?) open"

This confirms that Kali can establish a TCP connection to the Splunk receiver on port 9997.

An inverse host lookup failed message was also displayed. This relates to reverse hostname resolution and did not prevent the TCP connection from succeeding.

Evidence
![Kali to Splunk Connectivity](screenshots/01-kali-network/kali-to-splunk-connectivity.png)

4. Universal Forwarder Status 

The Splunk Universal Forwarder configuration on Kali was checked using:

sudo /opt/splunkforwarder/bin/splunk list forward-server

The forwarder reported:

Active forwards:
    172.20.10.2:9997

Configured but inactive forwards:
    None

This confirms that the Universal Forwarder recognizes the Splunk server at 172.20.10.2:9997 as an active forwarding destination.

The forwarder status screenshot is stored separately under:

![Splunk Universal Forwarder Status](screenshots/01-kali-network/forwarder-status.png)

5. Previous Splunk Analysis

Splunk Search & Reporting had previously been used to examine indexed events in the lab.

A preserved screenshot shows a previous Splunk search returning more than 75,000 events, including Splunk internal and macOS-related events.

This demonstrates previous use of Splunk for event searching and analysis.

It should not, however, be treated as evidence that Kali authentication logs were indexed because the visible events in the screenshot primarily originate from the Mac/Splunk environment.

Evidence

![Previous Splunk Analysis](screenshots/02-splunk/previous-splunk-analysis.png)


Current Status

The following components have been verified:

Splunk Enterprise is running.
TCP port 9997 is configured as a Splunk receiving input.
Kali can reach the Splunk server on TCP 9997.
The Kali Universal Forwarder reports 172.20.10.2:9997 as an active forward.
/var/log/auth.log on Kali is generating authentication and system activity.
Previous Splunk search activity has been preserved as project evidence.
