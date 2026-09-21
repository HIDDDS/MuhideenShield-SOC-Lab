# Kali Network Architecture

## Overview

This section documents the network configuration of the Kali Linux system used as the log source in the MuhideenShield SOC Lab.

Kali Linux is used to generate and collect authentication and system activity logs. The Splunk Universal Forwarder installed on Kali forwards selected logs to the Splunk Enterprise server for centralized monitoring and analysis.

## Kali Linux Network Configuration

| Configuration     | Value                           |
| ----------------- | ------------------------------- |
| Hostname          | `kali`                          |
| Network Interface | `eth0`                          |
| IP Address        | `172.20.10.5/28`                |
| Broadcast Address | `172.20.10.15`                  |
| Role              | Log source / monitored endpoint |

## Network Architecture

```text
Kali Linux
172.20.10.5/28
     |
     | Splunk Universal Forwarder
     | TCP 9997
     ↓
Splunk Enterprise
172.20.10.2
```

## Purpose

The Kali system acts as a monitored endpoint within the lab. Authentication and system activity generated on the endpoint can be collected and forwarded to Splunk.

This setup provides a controlled environment for practising:

* Authentication log analysis
* Failed and successful login investigation
* Security event detection
* Incident investigation
* SIEM monitoring
* SOC analyst workflows

## Evidence

The network configuration was verified using the following Linux commands:

```bash
hostname
ip addr
```

The resulting configuration is documented in the accompanying screenshot:

![Kali Network Configuration](../../screenshots/01-kali-network/kalinetworkconfig.png)

## Status

**Status:** Verified

The Kali endpoint is configured and ready to continue with Splunk Universal Forwarder and log forwarding configuration.
