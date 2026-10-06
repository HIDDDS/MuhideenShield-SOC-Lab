# Wazuh screenshot evidence

Original screenshots supplied during the lab, copied without image modifications. Capture times are filenames, not necessarily event times. The September 29 SSH/sudo/logout matches were reported in the conversation; no separate screenshot of those exact matches was supplied.

## Local authentication log inspection

Capture: 2026-09-27 at 01.39.28.

![Local authentication log inspection](01-local-auth-log.png)

## Sudo event in Wazuh

Capture: 2026-09-27 at 01.54.54.

![Sudo event in Wazuh](02-sudo-dashboard.png)

## SSH failure event list

Capture: 2026-09-27 at 10.44.46.

![SSH failure event list](03-failure-events.png)

## Related PAM failure details

Capture: 2026-09-27 at 10.46.29.

![Related PAM failure details](04-pam-failure-details.png)

## Successful loopback login in terminal

Capture: 2026-09-27 at 10.55.06.

![Successful loopback login in terminal](05-loopback-success-terminal.png)

## Successful loopback event in Wazuh

Capture: 2026-09-27 at 10.59.09.

![Successful loopback event in Wazuh](06-loopback-success-wazuh.png)

## Docker Desktop — Wazuh containers running

Capture: October 6, 2026 at 15:57:06 (supplied filename).

![Docker Desktop showing the three Wazuh containers running](docker-desktop-running-2026-10-06.png)

The Docker engine and the dashboard, manager, and indexer containers show running indicators. This image replaces the three older Docker screenshots and documents the October 6 state, not the September 29 outage or recovery. It does not by itself verify end-to-end log ingestion.

## Failed password with Kali as source

Capture: 2026-09-30 at 16.38.34.

![Failed password with Kali as source](10-kali-origin-failure.png)

## Dashboard before agent enrollment

Capture: September 26, 2026 at 11.05.30.

![Dashboard before agent enrollment](11-before-agent-enrollment.png)

## Manager connectivity and agent installation

Capture: September 26, 2026 at 11.34.41.

![Manager connectivity and agent installation](12-agent-installation.png)

## Kali endpoint dashboard

Capture: September 26, 2026 at 12.12.56.

![Kali endpoint dashboard](13-endpoint-dashboard.png)
