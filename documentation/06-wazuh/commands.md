# Commands used and what they mean

These commands document the lab workflow; run them in the stated environment. IPs, time windows, process IDs, and source ports must be adjusted to the event being investigated.

## Identify the shell before generating traffic

```bash
hostname
whoami
```

`hostname` identifies the computer executing commands. `whoami` identifies the command's effective user. A Mac Terminal tab with an SSH session can return `kali` because execution is remote.

On Mac, Command+T opens another Terminal tab. `ifconfig` displays interface addresses; a local `inet 172.20.10.2` line was checked against the SSH source. This only establishes the current address, not its historical owner.

## Connect from the local Mac shell

```bash
ssh kali@172.20.10.5
```

`ssh` establishes an encrypted connection; `kali` is the destination account, and the address identifies the server. Do not append the network prefix `/28` to the SSH destination.

For the controlled password test:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no kali@172.20.10.5
```

`-o` supplies an SSH option. These options select password authentication and disable public-key authentication for that connection. The exercise instructed one incorrect password, then Control+C at the next prompt. No automated password guessing was performed.

Within an established SSH session:

```bash
echo "$SSH_CONNECTION"
tty
```

`echo` displays the SSH connection variable: client IP, client port, server IP, server port. `tty` displays the current terminal, such as `/dev/pts/1`.

```bash
sudo whoami
exit
```

`sudo` runs the specified command as root by default, subject to account permissions. It does not bypass authorization or permanently change an ordinary shell's identity. `exit` closes the current shell; in our SSH session this returned the terminal to the Mac prompt.

## Read Kali logs

```bash
sudo tail -n 20 /var/log/auth.log
sudo journalctl -t sudo -n 10 --no-pager
sudo journalctl -u ssh --since "10 minutes ago" --no-pager
```

- `tail -n 20`: last 20 lines of a file.
- `journalctl`: reads the system journal.
- `-t sudo`: filters by syslog identifier/tag.
- `-n 10`: limits output to 10 recent entries.
- `-u ssh`: filters by service unit; some relevant records were missed by this filter in the lab.
- `--since`: limits the time window.
- `--no-pager`: prints without an interactive pager.

When a narrow query missed SSH evidence:

```bash
sudo journalctl --since "15 minutes ago" --no-pager | grep -Ei 'sshd|51080'
```

The pipe sends output into `grep`. `-E` enables the alternative pattern `|`; `-i` ignores letter case. The search keeps lines containing `sshd` or the specific source port. A source port is meaningful together with addresses and time, not as a globally unique session ID.

## Check the agent on Kali

```bash
sudo systemctl status wazuh-agent --no-pager
sudo tail -n 40 /var/ossec/logs/ossec.log
sudo cat /var/ossec/var/run/wazuh-agentd.state
```

These check service status, recent internal messages, and the agent state file. `cat` prints a file. Inspect the latest successful messages as well as errors. In this lab, restarting the agent corrected its displayed timezone:

```bash
sudo systemctl restart wazuh-agent
```

This interrupts monitoring briefly; it was followed by connection and fresh-event verification.

## Check the Docker host on Mac

```bash
cd /Users/muhideen/wazuh-docker/single-node
docker compose ps -a
docker context ls
docker info
docker ps -a
```

`cd` changes directories. Compose lists containers for the project; `docker ps -a` lists all containers, including stopped ones. Contexts identify Docker endpoints. `docker info` queries client/server information. A hung server query is different from a successful query showing stopped containers.

## Inspect Wazuh evidence

Select the Kali agent, use an appropriate time range, refresh results, and expand `full_log`. Compare account, source IP, source port, process ID, and event time with the original record. For yesterday's events, use an absolute date range instead of a short rolling window. Record raw timestamp and dashboard timezone separately when they differ.
