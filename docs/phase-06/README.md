# Phase 6 – Incident Response

## Phase 6 Objective

This phase investigates the security events generated during the attack simulations completed in Phase 5. The goal is to confirm that Splunk collected enough evidence in the Linux authentication logs to reconstruct what happened, identify the accounts and source IP addresses involved, and organize the findings the way an analyst would when reviewing a real incident.

## Investigation

### SSH Brute Force Investigation

Two separate IP addresses generated failed SSH login attempts against soc-server during testing: `10.0.2.2`, the original NAT test source used since earlier phases, and `192.168.56.102`, the kali-attacker VM introduced in Phase 5.

**Comparing failed and successful attempts by source IP**

```spl
index=main source="/var/log/auth.log" ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count(eval(searchmatch("Failed password"))) as Failed_Attempts count(eval(searchmatch("Accepted password"))) as Successful_Logins by src_ip
```

| src_ip | Failed_Attempts | Successful_Logins |
|---|---|---|
| 10.0.2.2 | 25 | 19 |
| 192.168.56.102 | 8 | 3 |

![Failed and accepted password events grouped by source IP](../../images/phase-06/02-failed-accepted-stats-by-ip.png)

This confirms both sources had a mix of failed and successful logins, but at different scales. 10.0.2.2 shows a much higher volume of activity, which lines up with it being the source used for earlier testing across multiple phases, while 192.168.56.102 reflects the more recent, smaller scale simulation run from kali-attacker.

**Failed attempts grouped by source IP**

```spl
index=main source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort -count
```

![Failed password count grouped by source IP](../../images/phase-06/04-failed-password-count-by-ip.png)

**Reviewing failed attempts from 10.0.2.2**

```spl
index=main source="/var/log/auth.log" "Failed password" | rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" | search src_ip="10.0.2.2"
```

![Failed password events from 10.0.2.2](../../images/phase-06/05-failed-password-src-10-0-2-2.png)

A number of these attempts were against `invalid user sam`, a username that does not exist on soc-server. This is a different account than `samm`, the real account on the server:

```
2026-07-29T04:37:07.449496+00:00 soc-server sshd-session[1743]: Failed password for invalid user sam from 10.0.2.2 port 62855 ssh2
```

**Reviewing failed attempts from 192.168.56.102**

```spl
index=main source="/var/log/auth.log" "Failed password" | rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" | search src_ip="192.168.56.102"
```

![Failed password events from 192.168.56.102](../../images/phase-06/06-failed-password-src-192-168-56-102.png)

This search returned 8 events, all against the valid account `samm`, unlike the invalid user attempts seen from 10.0.2.2.

**Full session detail for 192.168.56.102**

```spl
index=main source="/var/log/auth.log" "192.168.56.102"
| stats count by _raw
```

![Full raw event list for 192.168.56.102](../../images/phase-06/07-raw-session-events-192-168-56-102.png)

Looking at the raw events instead of just the failed password lines shows the full shape of the activity. On 8/5/26, there were two separate brute force attempts, session 24391 and session 24431, each ending the same way, with the SSH client itself closing the connection after the third failed attempt:

```
2026-08-05T07:57:26.689923+00:00 soc-server sshd-session[24391]: Connection closed by authenticating user samm 192.168.56.102 port 34040 [preauth]
2026-08-05T07:57:26.691533+00:00 soc-server sshd-session[24391]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.56.102 user=samm
```

The `[preauth]` tag and the `PAM 2 more authentication failures` line both confirm the session ended because of repeated failed logins, and the pattern of exactly 3 failures per session matches the default retry limit most SSH clients use before giving up.

**A second brute force attempt on 8/7/26**

```spl
index=main source="/var/log/auth.log" "Failed password"
```

![Three failed password events from session 1757](../../images/phase-06/01-failed-password-narrow-window.png)

![Same three failed password events with full detail](../../images/phase-06/08-failed-password-3events-aug7.png)

A second round of 3 failed attempts occurred on 8/7/26 between 7:53:44 AM and 7:53:53 AM, session 1757, again from 192.168.56.102 against the account `samm`. Shortly after, a successful login followed:

```
2026-08-07T07:54:18.265579+00:00 soc-server sshd-session[1762]: Accepted password for samm from 192.168.56.102 port 37322 ssh2
```

![Failed attempts followed by a successful login](../../images/phase-06/03-failed-accepted-events-30days.png)

**Investigation summary:** kali-attacker generated two separate SSH brute force attempts against the valid account `samm`, one on 8/5 and one on 8/7. Both attempts stopped at 3 failed logins per session, matching normal SSH client behavior rather than an automated tool retrying indefinitely. The account was never actually broken into through guessing. The 8/7 attempt was followed by a successful login on a new session once the correct password was used, which is consistent with testing rather than an actual compromise.

### Nmap Reconnaissance Investigation

```spl
index=main source="/var/log/auth.log" "Unable to negotiate"
```

![SSH negotiation failures generated by the nmap scan](../../images/phase-06/09-nmap-unable-to-negotiate-events.png)

This search returned 6 events, all from `192.168.56.102`, all within the same second on 8/5/26 around 8:34:49 AM:

```
2026-08-05T08:34:49.850810+00:00 soc-server sshd-session[29511]: Unable to negotiate with 192.168.56.102 port 57276: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group-exchange-sha256 [preauth]
```

These events happened because nmap's service detection scan connects to the SSH port and offers a fixed set of key exchange algorithms as part of fingerprinting the service, including older algorithms that soc-server's OpenSSH version no longer accepts. The connection fails during negotiation, before any username or password is ever sent. The `[preauth]` tag confirms this. No login was attempted and no credentials were involved, so this is not an intrusion.

**Investigation summary:** six negotiation failures from a single source, all within the same second, is a pattern that comes from an automated tool scanning the port rather than someone manually connecting. This confirms kali-attacker scanned soc-server around 8:34 AM on 8/5, matching the reconnaissance activity carried out in Phase 5. It does not represent unauthorized access, but it is useful supporting evidence that a scan took place at that time.

### Privilege Escalation Investigation

```spl
index=main source="/var/log/auth.log" sudo
```

![Sudo events over the last 30 days](../../images/phase-06/10-sudo-events-30days.png)

This search returned 241 events over 30 days. The events reviewed show routine administrative activity carried out by `samm`, including switching to the splunk service account and shutting the server down:

```
2026-08-10T23:50:39.254324+00:00 soc-server sudo: samm : TTY=/dev/tty1 ; PWD=/home/samm ; USER=root ; COMMAND=/usr/bin/su - splunk
2026-08-07T08:26:05.446637+00:00 soc-server sudo: samm : TTY=/dev/tty1 ; PWD=/home/samm ; USER=root ; COMMAND=/usr/sbin/shutdown now
```

**Investigation summary:** the sudo activity reviewed here is normal administrative use by `samm`, not a sign of privilege escalation or misuse. No brute force pattern against sudo was found in these events.

## Timeline

Every timestamp below is taken directly from the Splunk searches above.

| Time | Event |
|---|---|
| 8/5/26 7:39:53 AM | Accepted password for samm from 192.168.56.102 |
| 8/5/26 7:50:25 AM | Session disconnected by user samm, 192.168.56.102 |
| 8/5/26 7:57:14 – 7:57:26 AM | 3 failed password attempts from 192.168.56.102 (session 24391), connection closed by client after the third failure |
| 8/5/26 7:57:32 – 7:57:34 AM | Additional failed password attempts from 192.168.56.102 (session 24431) |
| 8/5/26 8:34:49 AM | 6 SSH negotiation failures from 192.168.56.102, generated by the nmap scan |
| 8/7/26 7:53:44 – 7:53:53 AM | 3 failed password attempts from 192.168.56.102 (session 1757) |
| 8/7/26 7:54:18 AM | Accepted password for samm from 192.168.56.102 (session 1762) |

A broader search across the full `auth.log` source, sorted by time, returned 907 events between 7/12/26 and 8/11/26, confirming the log source retains enough history to cover every event referenced above.

![Full auth.log source sorted by time, 907 events](../../images/phase-06/11-full-authlog-907events.png)

## Evidence

| Screenshot | What It Shows |
|---|---|
| `01-failed-password-narrow-window.png` | Failed password events from session 1757 in a narrow time window |
| `02-failed-accepted-stats-by-ip.png` | Failed and accepted login counts grouped by source IP |
| `03-failed-accepted-events-30days.png` | Failed and accepted events over 30 days, including the successful login that followed the 8/7 brute force attempt |
| `04-failed-password-count-by-ip.png` | Failed password count grouped by source IP |
| `05-failed-password-src-10-0-2-2.png` | Failed password events specific to 10.0.2.2, including attempts against an invalid user |
| `06-failed-password-src-192-168-56-102.png` | Failed password events specific to 192.168.56.102 |
| `07-raw-session-events-192-168-56-102.png` | Full raw event list for 192.168.56.102, showing session boundaries and disconnects |
| `08-failed-password-3events-aug7.png` | The 3 failed password events from the 8/7 brute force attempt |
| `09-nmap-unable-to-negotiate-events.png` | SSH negotiation failures generated by the nmap scan |
| `10-sudo-events-30days.png` | Sudo activity over 30 days |
| `11-full-authlog-907events.png` | Full auth.log source sorted by time |

## MITRE ATT&CK Mapping

| Activity | Technique | ID |
|---|---|---|
| SSH Brute Force | Brute Force | T1110 |
| SSH Login | Remote Services: SSH | T1021.004 |
| Successful Authentication | Valid Accounts | T1078 |
| Network Reconnaissance | Active Scanning | T1595 |

## Lessons Learned

Searching for the exact phrase "Failed password" first, then breaking the results down by source IP with `stats`, made it possible to separate the two different sources of activity without confusing them. Repeating the same style of search on 8/7 confirmed the brute force pattern was repeatable and not a one time result from earlier testing.

Looking at the full raw event list for a specific IP, instead of only the failed password lines, showed details that a narrower search would have hidden, including the disconnect events and the exact point where the SSH client gave up after 3 attempts. This made it clear the brute force attempts were not open ended, they stopped on their own.

Reviewing the sudo activity also showed why context matters when a search returns a large number of events. 241 events is a lot to look at individually, but comparing a few sample events against what was expected from normal administrative work was enough to confirm nothing unusual was happening there.

## Conclusion

This investigation confirmed that Splunk successfully collected and correlated the Linux authentication events generated during the attack simulation. The SSH brute force attempts, the successful logins that followed, and the SSH negotiation failures caused by the nmap scan were all present in `auth.log` with accurate timestamps and source IP addresses, and the same searches produced consistent results when run again days later. The logging pipeline built in earlier phases held up under real, generated attack traffic and provided enough detail to reconstruct exactly what happened and when.
