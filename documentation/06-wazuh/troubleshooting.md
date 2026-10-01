# Wazuh troubleshooting record

## 1. Docker and dashboard unresponsive — September 29

**Symptoms:** The dashboard stopped loading. `docker compose ps -a` hung in the single-node directory, and `docker info` displayed client information before stalling at the server section. Commands were cancelled with Control+C. Docker Desktop showed its engine running but an empty container list.

**Checks:** `docker context ls` responded and identified `desktop-linux` as active. An OrbStack context also existed; this did not establish a conflict. Quitting and reopening Docker Desktop did not resolve the issue.

**Recovery:** After saving work and shutting down Kali normally, the Mac was restarted. Docker Desktop was opened, then `docker ps -a` returned the manager, indexer, and dashboard containers as running. The dashboard at `https://localhost` loaded again. Kali was started, the agent was checked, and a fresh `sudo whoami` event was located in Wazuh.

**Result:** Service and fresh ingestion restored. The root cause was not established. No factory reset, volume purge, or data deletion was used. Neither memory exhaustion nor an OrbStack conflict was proven.

Docker resource allocation had previously been reduced from 6 GB to 4 GB on an 8 GB Mac. This is configuration context, not proof of the outage cause.

## 2. Timezone name rejected

`Africa/lagos` was rejected as invalid. Correcting the case to `Africa/Lagos` worked:

```bash
sudo timedatectl set-timezone Africa/Lagos
```

This sets the display timezone. It does not by itself verify clock synchronization with a time source.

## 3. SSH events missed by a filter

An early `journalctl -t sshd` query showed listener messages but missed authentication records tagged `sshd-session`. Later, queries using `-u ssh` and 10- and 30-minute windows returned no entries for the logout. A broader search found the records:

```bash
sudo journalctl --since "15 minutes ago" --no-pager | grep -Ei 'sshd|51080'
```

The evidence was present outside the results of the narrower filter. The exact systemd unit association was not investigated. Broaden the query when a filter returns no results; do not immediately conclude that the activity never occurred.

## 4. Agent running, but recent dashboard activity uncertain

The dashboard's most recent observed item was around 16:40 and was described as a disconnect. The agent service was still `active (running)`.

```bash
sudo tail -n 40 /var/ossec/logs/ossec.log
sudo cat /var/ossec/var/run/wazuh-agentd.state
```

The log contained old TCP 1514 and enrollment TCP 1515 errors, followed by `Connected to the server` and `Agent is now online`. The state file reported:

```text
status='connected'
last_keepalive='2026-09-29 12:27:04'
last_ack='2026-09-29 12:27:04'
msg_buffer='0'
```

The local clock displayed current Lagos time, approximately 17:27. A retained process timezone was suspected. After restarting the agent:

```bash
sudo systemctl restart wazuh-agent
sudo cat /var/ossec/var/run/wazuh-agentd.state
```

the output included:

```text
status='connected'
last_keepalive='2026-09-29 17:29:23'
last_ack='2026-09-29 17:29:23'
msg_buffer='0'
```

A new `sudo whoami` event appeared in Wazuh with the expected current time. The September 29 SSH closure was subsequently confirmed in Wazuh on September 30.

**Verified:** Restart corrected the displayed agent timestamps and a fresh event reached the dashboard. **Unresolved:** The original reason the earlier closure was not visible, and whether it was delayed, outside the searched window, or missed during inspection. We did not establish event loss or that the timezone difference caused an ingestion failure.

## 5. Wrong terminal context — September 30

The failed-login source was `172.20.10.5`, not the expected Mac address `.2`. `hostname` returned `kali`. Opening a new Mac Terminal tab and checking `hostname` returned the Mac hostname. An SSH terminal displayed on a Mac still executes commands on the remote system. The corrected test remains pending.

## Monitoring checks are different

| Check | What it establishes |
|---|---|
| Service active | A process is running |
| Agent connected / recent acknowledgement | Agent-manager communication is established |
| Empty agent buffer | No events currently waiting in that buffer |
| Matching fresh event in dashboard | That specific event completed the collection and indexing path |

None of these alone proves every expected log source is collected without loss.
