# Controlled SSH and sudo investigation

**Period:** September 26–30, 2026. **Classification:** Authorized lab activity. **Status:** September 29 workflow verified; September 30 Mac-origin failure/success comparison unfinished.

## Objective

Follow authentication activity from a Linux endpoint into Wazuh, distinguish login identity from command execution identity, and build an evidence-based timeline. This is a practice investigation, not a report of a real intrusion.

## Initial loopback tests, September 26–27

Kali initially acted as both SSH client and server. Password authentication was tested with:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no kali@127.0.0.1
```

Local records showed failed passwords at September 26 21:10:36 and 21:10:51 from port `56132`, and 21:15:55 from port `37460`. These are original displayed local times, before the timezone correction. Wazuh showed three failed-password rows and related PAM events. Several messages can describe one authentication attempt, so PAM and password messages must not be counted as separate attempts automatically.

The logs also contained `pam_winbind` availability errors. Those errors were not diagnosed or fixed. Authentication records appeared under `sshd-session`, so filtering only the `sshd` tag initially missed them.

A later successful loopback event contained:

```text
Accepted password for kali from 127.0.0.1 port 40990 ssh2
```

The Wazuh screenshot matched process `20706`, account `kali`, source `127.0.0.1`, and source port `40990`. The terminal displayed September 27 05:53:25, while Wazuh raw text and dashboard displayed different hours. These times must not be combined without timezone normalization. The failures and success were hours apart; this did not validate rapid brute-force correlation.

## Mac-to-Kali investigation, September 29

The Mac initiated SSH to `kali@172.20.10.5`. Within that session:

```bash
echo "$SSH_CONNECTION"
```

returned:

```text
172.20.10.2 51080 172.20.10.5 22
```

The fields identify client IP, client port, server IP, and server port. The source port is temporary; port 22 is the server's listening port. The matching accepted-password event was confirmed in Wazuh by the lab operator, but its exact timestamp was not copied into the session notes.

| Time, September 29 | Activity | Verification |
|---|---|---|
| Exact login time not captured | `kali` authenticated from `172.20.10.2:51080` | Session output and user-confirmed Wazuh match |
| 16:08:50, Kali displayed time | `sudo whoami`, process `5572`, `TTY=pts/1` | Pasted journal record and user-confirmed Wazuh match |
| 16:10:26, Kali displayed time | Sudo used to read the journal | Pasted journal record |
| 17:06:38, Kali displayed time | Client disconnected, SSH session closed | Pasted local record; Wazuh match confirmed the next day |

### Privileged command evidence

```text
Sep 29 16:08:50 kali sudo[5572]: kali : TTY=pts/1 ; PWD=/home/kali ; USER=root ; COMMAND=/usr/bin/whoami
Sep 29 16:08:50 kali sudo[5572]: pam_unix(sudo:session): session opened for user root(uid=0) by kali(uid=1000)
Sep 29 16:08:50 kali sudo[5572]: pam_unix(sudo:session): session closed for user root
```

`kali` requested the command; `root` was its execution identity. This does not show an SSH login as root. The terminal outputs were `kali` for `whoami` and `root` for `sudo whoami`. The ordinary SSH shell remained under `kali`.

The `tty` command returned `/dev/pts/1`, supporting the connection between the current terminal and sudo record. Terminal numbers can be reused; this is supporting context, not an immutable historical session identifier. The sudo record names the invoked command, not its output or every command executed in the shell.

### Logout evidence

```text
Sep 29 17:06:38 kali sshd-session[5103]: Received disconnect from 172.20.10.2 port 51080:11: disconnected by user
Sep 29 17:06:38 kali sshd-session[5095]: syslogin_perform_logout: logout() returned an error
Sep 29 17:06:38 kali sshd-session[5103]: Disconnected from user kali 172.20.10.2 port 51080
Sep 29 17:06:38 kali sshd-session[5095]: pam_unix(sshd:session): session closed for user kali
```

The source IP and port match the earlier session. The logout bookkeeping error was observed but not diagnosed; separate records explicitly show disconnection and PAM session closure. The SSH closure is different from the sudo session closing after a command.

## September 30: checking the actual source

The new exercise intended one wrong password from the Mac, followed later by a correct login. The screenshot instead showed:

```text
Sep 30 15:32:36 kali sshd-session[19648]: Failed password for kali from 172.20.10.5 port 52286 ssh2
```

The source matched Kali's own address. Running `hostname` in the originating terminal returned `kali`, confirming that commands were executing on Kali. A new Mac Terminal tab returned `Hammeds-MacBook-Air.local`.

![Failed-password event with Kali as source](../screenshots/05-wazuh/10-kali-origin-failure.png)

The raw event timestamp was 15:32:36; the dashboard list displayed events around 16:32. The one-hour difference has not been fully investigated, so the raw time is retained without an assumed timezone conversion.

The corrected Mac-origin failure test was instructed, but its result was not supplied before documentation began. A subsequent successful Mac login for this exercise has also not been validated. Neither a completed failure-to-success sequence nor an automated correlation detection is claimed for September 30.

## Analyst assessment

The tested activity was expected because I generated it and verified its context. Successful authentication establishes that credentials were accepted, not that the rightful owner used them. A failed password alone does not establish compromise; its significance depends on the account, source, timing, frequency, and surrounding behavior.

In production, I would check time-specific DHCP/VPN mappings, asset ownership, account-owner confirmation, and endpoint activity. A private source IP cannot be attributed to a person using public geolocation. Those enterprise evidence sources were discussed conceptually, not deployed in this lab.

## Remaining evidence gaps

- Capture the exact September 29 accepted-login timestamp with its timezone.
- Verify the new Mac-origin failed and successful login records.
- Normalize raw event time versus dashboard display time.
- Test and document custom correlation logic separately before claiming detection validation.
- Expand post-login telemetry; current sudo evidence is not full command or process auditing.
