# Lessons from SSH, sudo, and Wazuh

## Technical lessons

- SSH connects a client to a server. Roles apply to a connection: Kali can connect to itself, or the Mac can initiate a connection to Kali.
- `127.0.0.1` means loopback. Connecting to Kali's own interface address can also produce Kali-to-Kali traffic.
- `agent.ip` identifies the monitored endpoint; `data.srcip` identifies the source extracted from the event. They answer different questions.
- A terminal window's physical location does not determine where commands execute. Check `hostname`.
- `USER=root` in a sudo record identifies the target execution account; it does not establish an SSH login as root.
- Sudo session closure and SSH session closure are distinct events.
- One authentication attempt can generate several messages. Count the appropriate events, not every related PAM line.
- Process running, agent connected, and event searchable are separate validation stages.
- Timezones, query filters, and time windows can hide relevant evidence.

## Analyst reasoning

Successful authentication does not establish legitimacy. Failed authentication does not automatically establish an attack. I practised checking account, source, time, sequence, expected activity, and user confirmation before drawing a conclusion.

In an organization, staff devices do not send logs simply because they join Wi-Fi. The team configures agents or other collection methods, network access, and log sources. Device logs, network logs, and identity logs provide different visibility. DHCP lease history can help map an IP to a device at the event time; inventory and endpoint records can help identify its assigned owner or active user. Ownership alone does not prove who initiated an action.

Those enterprise workflows were discussed, not implemented. No staff monitoring, corporate network collection, Mac agent installation, or EDR deployment was completed here.

## Learning approach

I practised explaining command options, predicting outputs, comparing raw logs with parsed fields, and correcting an incorrect interpretation of sudo. Guided execution is part of this learning record. My next goal is to repeat a small investigation independently and explain what each piece of evidence proves and what remains uncertain.

## Next work

1. Finish the Mac-origin failed-password test and subsequent successful login.
2. Build a timestamped comparison using actual records and confirmed timezones.
3. Write a short triage assessment with evidence, alternative explanations, and gaps.
4. Implement and validate a custom detection separately; choose and test any thresholds explicitly.
5. Add another telemetry source only after the current investigation is understood.

Private lab IPs and terminal IDs are useful practice evidence; they are not permanent device or person identifiers.
