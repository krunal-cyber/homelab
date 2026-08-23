# Lab: Centralized Logging with rsyslog

## Objective

Configure centralized Linux logging using `rsyslog` so that logs generated on a Parrot OS client are forwarded over UDP to an Ubuntu central logging server.

The lab also focused on understanding Linux logging components, troubleshooting network and logging issues, and verifying the complete log-delivery pipeline.

## Lab Environment

### Client
- OS: Parrot OS
- Role: Log client / forwarder
- Lab IP: `192.168.20.11`

### Central Server
- OS: Ubuntu
- Role: Centralized log server
- Lab IP: `192.168.20.1`

### Network
- Lab subnet: `192.168.20.0/25`
- Syslog transport: UDP
- Syslog port: `514`

A temporary VMware NAT adapter was used on Parrot to obtain Internet access for installing packages. Lab communication between Parrot and Ubuntu remained on the isolated `192.168.20.0/25` network.

## 1. Centralized Logging

Centralized logging collects events from multiple systems at a central location.

```text
Multiple systems
      |
      v
Central Log Server
      |
      v
SIEM / Monitoring
      |
      v
SOC Analyst
```

This improves searching, correlation, alerting, and incident investigation.

## 2. Important Components

### `logger`

`logger` is a command-line utility used to generate a syslog message.

```bash
logger "CENTRALIZED-LOG-TEST"
```

It was used to generate test events. It does not automatically record every command executed by a user.

### `rsyslog`

`rsyslog` is a Linux logging service/daemon that can receive, process, filter, store, and forward log messages.

In this lab:
- Parrot used rsyslog as a log forwarder.
- Ubuntu used rsyslog as the centralized collector.

### `imuxsock`

`imuxsock` is an rsyslog input module for receiving local syslog messages through a Unix socket, commonly `/dev/log`.

```text
logger
   |
   v
/dev/log
   |
   v
imuxsock
   |
   v
rsyslog
```

A Unix socket is a local inter-process communication endpoint, not a normal log file.

### `imjournal`

`imjournal` is an rsyslog input module that reads messages from `systemd-journald`.

```text
system/application
       |
       v
systemd-journald
       |
       v
imjournal
       |
       v
rsyslog
```

The systemd journal can contain system-generated events as well as application/user-generated messages.

### `imudp`

`imudp` is an rsyslog input module that receives syslog messages over UDP.

Ubuntu was configured to listen on UDP port `514`.

## 3. Final Architecture

```text
                    PARROT
                 192.168.20.11
                       |
                     logger
                       |
                    /dev/log
                       |
                    imuxsock
                       |
                    rsyslog
                       |
                    UDP 514
                       |
                       v
                    UBUNTU
                 192.168.20.1
                       |
                    imudp
                       |
                    rsyslog
                       |
                  RemoteLogs
                       |
              /var/log/remote/
```

## 4. Procedure

### Step 1 — Install and prepare rsyslog on Parrot

The initial `rsyslog` installation failed because APT could not reach the Parrot repositories.

`apt update` showed repository/DNS connectivity problems.

Parrot was originally on an isolated lab network, so a temporary VMware NAT adapter was added for package installation. The default route was configured to use NAT for Internet access while retaining the direct route to the lab subnet.

The lab subnet remained:

```text
192.168.20.0/25
```

A default route was not required for Parrot to communicate with Ubuntu because both were on the same directly connected subnet.

### Step 2 — Configure Parrot

The forwarding rule was:

```text
*.* @192.168.20.1:514
```

This means all facilities/severities are forwarded to Ubuntu using UDP.

The local input module was configured as:

```text
module(load="imuxsock")
```

Configuration was validated with:

```bash
sudo rsyslogd -N1
```

and rsyslog was restarted with:

```bash
sudo systemctl restart rsyslog
```

### Step 3 — Configure Ubuntu

Ubuntu was configured to receive remote syslog:

```text
module(load="imudp")
input(type="imudp" port="514")
```

A remote log directory was created:

```bash
sudo mkdir -p /var/log/remote
```

A dynamic template and action were configured:

```text
template(name="RemoteLogs" type="string" string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")

*.* ?RemoteLogs
& stop
```

The template defines the destination format, while `*.* ?RemoteLogs` tells rsyslog to use that template for writing messages.

## 5. Troubleshooting

### Issue 1 — `rsyslog` could not be installed

APT reported that `rsyslog` had no installation candidate.

**Cause:** Parrot could not reach the package repository because it was on an isolated network without Internet access.

**Resolution:** A temporary NAT adapter and correct Internet routing were configured.

### Issue 2 — DNS failed

Repository hostnames could not initially be resolved.

The problem was investigated using `/etc/resolv.conf` and connectivity tests. The underlying issue was network/default-route connectivity rather than simply the DNS server.

### Issue 3 — Default route concern

Removing the default route from the lab interface did not prevent communication with Ubuntu.

The route:

```text
192.168.20.0/25
```

was directly connected, so Ubuntu could be reached without a default gateway. The NAT interface was used for Internet traffic.

### Issue 4 — Parrot was not initially forwarding `logger` messages

The network path was tested independently using `tcpdump` and a direct UDP test.

Direct UDP traffic to:

```text
192.168.20.1:514
```

worked, proving that the network and UDP port were reachable.

The problem was therefore narrowed to Parrot's rsyslog input path.

The local input was configured with:

```text
module(load="imuxsock")
```

After restarting rsyslog, `logger` messages generated UDP/514 traffic.

### Issue 5 — Ubuntu received UDP packets but no log file was created

Ubuntu's `tcpdump` confirmed packets were arriving.

The rsyslog configuration contained the remote template, but the action rule required to use that template was missing.

The action was added:

```text
*.* ?RemoteLogs
& stop
```

### Issue 6 — Permission denied

After the action was added, rsyslog attempted to create remote files but could not write to `/var/log/remote`.

The directory was owned by root while rsyslog runs under the `syslog` account.

Permissions were corrected:

```bash
sudo chown syslog:adm /var/log/remote
sudo chmod 755 /var/log/remote
```

Rsyslog was restarted afterward.

## 6. Verification

A final test message was generated on Parrot:

```bash
logger "CENTRALIZED-LOG-FINAL-TEST"
```

Network traffic was verified with:

```bash
sudo tcpdump -ni any udp port 514
```

Ubuntu received the packet.

The final log was verified with:

```bash
sudo grep -R "CENTRALIZED-LOG-FINAL-TEST" /var/log/remote
```

The message was successfully found, proving the complete pipeline.

## 7. Filtering and Centralized Monitoring

Filtering can be performed on the client or centrally.

If a client filters out a security-relevant event before forwarding it, the central server cannot recover that event because it never received it.

Therefore, centralized environments generally collect a sufficiently broad set of security-relevant events at the endpoints and perform more advanced filtering, correlation, alerting, and detection centrally.

## 8. Monitoring Different Sources

Rsyslog does not automatically record every command executed by a user.

Commands such as:

```text
ls
cat
nano
ping
curl
```

normally do not become syslog events simply because rsyslog is running.

Security monitoring therefore uses multiple sources:

| Source | Purpose |
|---|---|
| systemd-journald | System and application journal events |
| rsyslog | Log processing, storage, and forwarding |
| auditd | Security auditing and process/file activity |
| sudo | Privileged command activity |
| SSH | Authentication events |
| Firewall | Network connection events |
| Web server | HTTP/application activity |

For example, `auditd` can be configured to monitor security-relevant command execution and file access, while rsyslog can collect and forward applicable log events.

## 9. Troubleshooting Method Used

The troubleshooting process isolated each layer instead of changing configurations randomly:

```text
1. Internet/DNS
       |
2. IP routing
       |
3. Local network connectivity
       |
4. UDP/514 connectivity
       |
5. logger
       |
6. rsyslog input
       |
7. rsyslog forwarding
       |
8. Ubuntu rsyslog listener
       |
9. Storage/action rule
       |
10. File permissions
```

`tcpdump` was particularly useful because it allowed the network path to be verified independently of rsyslog.

## 10. Key Learning Points

1. `logger` is a tool for generating syslog test messages.
2. `/dev/log` is a Unix socket, not a normal log file.
3. `imuxsock` allows rsyslog to receive local syslog messages through `/dev/log`.
4. `imjournal` allows rsyslog to read messages from systemd-journald.
5. `imudp` allows rsyslog to receive remote UDP syslog messages.
6. `rsyslog` processes, filters, stores, and forwards logs.
7. UDP/514 is commonly used for syslog forwarding.
8. A template defines how/where logs are stored, but an action rule must use the template.
9. File permissions can prevent rsyslog from creating log files.
10. `tcpdump` helps determine whether a problem is networking or logging configuration.
11. A directly connected subnet does not require a default route for communication within that subnet.
12. Centralized logging allows security events from multiple systems to be investigated in one location.
13. Rsyslog alone does not provide complete command auditing; tools such as `auditd` provide additional security telemetry.
14. SOC monitoring normally combines multiple sources rather than relying on one logging source.

## Conclusion

The centralized logging lab successfully configured Parrot OS as a log-forwarding client and Ubuntu as a centralized rsyslog server.

A test event successfully travelled through:

```text
logger
   |
/dev/log
   |
imuxsock
   |
Parrot rsyslog
   |
UDP/514
   |
Ubuntu imudp
   |
Ubuntu rsyslog
   |
RemoteLogs
   |
/var/log/remote/
```

The lab provided practical experience with Linux logging, rsyslog input modules, UDP troubleshooting, packet capture, routing, file permissions, centralized log storage, and the foundations of SOC monitoring.
