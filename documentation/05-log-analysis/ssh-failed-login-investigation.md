# SSH Failed Login Investigation

## Investigation Overview

This investigation was conducted as part of my **MuhideenShield SOC Lab** to practise authentication-log monitoring, SIEM analysis, and basic incident investigation.

The objective was to intentionally generate failed SSH authentication activity on my Kali Linux endpoint, forward the resulting authentication logs to Splunk, and investigate the activity using Splunk Search & Reporting.

This was performed in a controlled lab environment.

---

## Lab Environment

| Component             | Configuration              |
| --------------------- | -------------------------- |
| Monitored Endpoint    | Kali Linux                 |
| Kali IP Address       | `172.20.10.5`              |
| SIEM                  | Splunk Enterprise          |
| Splunk Server         | `172.20.10.2`              |
| Log Forwarder         | Splunk Universal Forwarder |
| Splunk Receiving Port | TCP `9997`                 |
| Authentication Log    | `/var/log/auth.log`        |
| Test Username         | `malicious_hacker`         |

---

## 1. Preparing the SSH Service

The SSH service was started on Kali Linux using:

```bash
sudo systemctl start ssh
```

I then checked the SSH service exposure using Nmap:

```bash
nmap -sS -p 22,80,443 localhost
```

The scan showed that SSH was accessible on:

```text
22/tcp open ssh
```

This confirmed that the SSH service was available for the controlled authentication test.

---

## 2. Generating Failed SSH Authentication

To generate authentication events for analysis, I attempted to connect to the SSH service using a deliberately invalid username:

```bash
ssh malicious_hacker@localhost
```

Incorrect credentials were supplied during the controlled test.

The authentication attempt returned:

```text
Permission denied, please try again.
```

This generated failed authentication activity that could be recorded by the Linux authentication logging system.

### Evidence

![Controlled SSH Failed Login](../../screenshots/04-log-analysis/ssh-failed-login-generation.png)

---

## 3. Authentication Log Collection

Kali records authentication-related activity in:

```text
/var/log/auth.log
```

The Splunk Universal Forwarder on the Kali endpoint was configured to communicate with the Splunk Enterprise server.

The forwarding destination had previously been verified using:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

which reported:

```text
Active forwards:
    172.20.10.2:9997

Configured but inactive forwards:
    None
```

This provided the forwarding path between the monitored Kali endpoint and the Splunk server.

---

## 4. Searching for the Activity in Splunk

After generating the failed SSH authentication activity, I searched Splunk for events associated with the test username.

The search used was:

```spl
index=* "malicious_hacker"
```

Splunk returned events containing the username.

The event metadata visible during the investigation included:

```text
host = kali
source = /var/log/auth.log
sourcetype = auth-2
```

This demonstrated that authentication events originating from the Kali system's `/var/log/auth.log` were available for investigation in Splunk.

### Evidence

![Kali Authentication Logs in Splunk](../../screenshots/04-log-analysis/splunk-kali-auth-search.png)

---

## 5. Investigating the Failed Authentication

The returned events contained authentication messages including:

```text
Invalid user malicious_hacker
```

and:

```text
Failed password for invalid user malicious_hacker
```

These events indicate an SSH authentication attempt using a username that was not valid on the system, followed by a failed password authentication attempt.

### Evidence

![Failed SSH Authentication Events](../../screenshots/04-log-analysis/splunk-failed-ssh-events.png)

---

## 6. Investigation Findings

The investigation established the following sequence:

```text
SSH service enabled on Kali
        ↓
Port 22 verified as open
        ↓
Controlled SSH login attempted
        ↓
Invalid username used
        ↓
Authentication failed
        ↓
Event recorded in /var/log/auth.log
        ↓
Authentication data available in Splunk
        ↓
Events identified through Splunk search
```

The activity was intentionally generated as part of the lab and therefore does not represent an actual compromise.

However, similar events in a production environment could warrant further investigation, particularly when repeated attempts originate from the same external source or target multiple accounts.

---

## 7. Analyst Interpretation

A single failed authentication attempt does not by itself establish malicious activity. Users can enter incorrect usernames or passwords legitimately.

However, patterns such as:

* repeated failed authentication attempts;
* attempts against multiple usernames;
* authentication attempts from unusual source addresses;
* invalid-user attempts;
* a successful login following numerous failures;

can increase the significance of the activity and provide useful indicators during an investigation.

A SOC analyst could correlate these events with additional authentication, endpoint, firewall, and network telemetry before determining whether escalation is required.

---

## 8. Recommended Monitoring

Based on this investigation, useful monitoring could include:

* Tracking failed SSH authentication attempts.
* Identifying repeated failures from the same source IP.
* Monitoring attempts involving invalid users.
* Comparing failed and successful authentication events.
* Creating thresholds for unusually high numbers of failures.
* Investigating successful authentication following repeated failures.

---

## 9. Skills Practised

This investigation provided hands-on practice with:

* Linux authentication logs
* SSH
* Nmap
* Splunk Enterprise
* Splunk Universal Forwarder
* SIEM searching
* Log analysis
* Authentication-event investigation
* Evidence collection
* SOC investigation documentation

---

## Conclusion

This lab demonstrated a basic end-to-end SOC workflow: generating controlled authentication activity on a monitored endpoint, locating the resulting authentication events in Splunk, analysing the events, and documenting the findings.

The next stage of the MuhideenShield SOC Lab will build on this investigation by developing a Splunk detection for repeated failed SSH authentication attempts and using that detection in a structured incident-investigation workflow.
