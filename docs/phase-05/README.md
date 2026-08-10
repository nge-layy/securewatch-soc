# Phase 5 – Attack Simulation & Lab Expansion

## Phase Objective

Phases 1–4 built a hardened, logged, and monitored single-node SOC — but a detection rule that has only ever been tested against synthetic or single-source data isn't proven yet. Phase 5 expands the lab into a small multi-host environment and uses it to generate **real security events**: a genuine reconnaissance scan, a genuine baseline login, and a genuine brute-force pattern, all captured by Splunk exactly as they'd appear from an actual attacker on the network. The goal is to validate the logging and detection pipeline built in earlier phases against activity that wasn't hand-crafted to match it.

## Section 1 – Lab Expansion

The lab now includes three virtual machines, each with a distinct role:

| VM | Role | Purpose in This Phase |
|---|---|---|
| `soc-server` (SecureWatch-Ubuntu-SOC) | Ubuntu Server running Splunk Enterprise | The monitored target — the host being scanned and logged against |
| `kali-attacker` | Kali Linux | The attacker-simulation host — used to run reconnaissance and SSH brute-force activity against `soc-server` |
| `win-client` | Windows client | Lab endpoint reserved for future Windows log collection (see Section 8) |

All three VMs communicate over the lab's internal networking (Host-Only and NAT adapters, established during earlier lab-expansion work), which is what allows `kali-attacker` to reach `soc-server` directly by IP address rather than through the host machine.

**IP addressing confirmed during this phase's testing:**

| Host | IP Address | Confirmed By |
|---|---|---|
| `soc-server` | `192.168.56.101` | Target of the Nmap scan in Section 5 |
| `kali-attacker` | `192.168.56.102` | Source IP recorded on every SSH event in Splunk during this phase |

This is the first phase where `kali-attacker` is used for its actual purpose — generating traffic against `soc-server` — rather than just being provisioned and network-configured.

## Section 2 – Network Verification

Before treating any SSH or Nmap activity as meaningful, basic reachability and service availability from `kali-attacker` to `soc-server` were confirmed directly through the Nmap scan itself, since a live port/service scan is a strong combined test of both connectivity and service state.

```bash
nmap -sV 192.168.56.101
```

**Why this command was used:** `nmap -sV` performs a TCP port scan against the target and additionally attempts to fingerprint the exact service and version listening on each open port. Running this from `kali-attacker` toward `soc-server` verifies, in a single step, that the two hosts can communicate on the network **and** confirms which services are actually reachable and what they identify themselves as.

**Result — ports found open on `soc-server` (192.168.56.101):**

| Port | State | Service | Version |
|---|---|---|---|
| 22/tcp | open | ssh | OpenSSH 10.2p1 Ubuntu 2ubuntu3.5 |
| 5432/tcp | open | postgresql | PostgreSQL DB 17.0 – 17.8 |
| 8000/tcp | open | http | Splunkd httpd |
| 8089/tcp | open | ssl/http | Splunkd httpd |

This single result confirms several things at once: the host is reachable on the network, SSH is available (needed for Sections 3–4), and the Splunk web (`8000`) and management (`8089`) ports are visible from `kali-attacker` — meaning `soc-server`'s Splunk instance is exposed on this network segment, not just on `localhost`.

![Nmap scan from kali-attacker against soc-server](docs/phase-05/08-nmap-scan-kali-terminal.png)
*Screenshot placement: terminal on `kali-attacker` running `nmap -sV 192.168.56.101` followed by `sudo nmap -A 192.168.56.101`, confirming network reachability and enumerating the open SSH, PostgreSQL, and Splunk services on `soc-server`.*

## Section 3 – SSH Baseline Test

Before simulating any attack, a normal, successful SSH login from `kali-attacker` to `soc-server` was performed and confirmed in Splunk.

**Why test a normal login first:** A detection rule is only meaningful if it's compared against a known-good baseline. Confirming what a *legitimate* login looks like in the logs — correct source IP, expected `Accepted password` event, clean session open/close — establishes the pattern that later brute-force activity should visibly deviate from. It also confirms the logging pipeline (established in Phase 3) is correctly capturing events from this new source host before any attack traffic is generated.

**Observed events in `auth.log` / Splunk:**

```
sshd-session[1739]: Accepted password for samm from 192.168.56.102 port 40128 ssh2
sshd-session[1739]: pam_unix(sshd:session): session opened for user samm(uid=1000) by samm(uid=0)
sshd-session[1699]: Connection closed by 192.168.56.102 port 55580
sshd[1342]: Server listening on :: port 22.
sshd[1342]: Server listening on 0.0.0.0 port 22.
```

![Baseline successful SSH login from Kali](../../images/phase-05/10-ssh-baseline-accepted-login.png)
*Screenshot placement: Splunk search results showing the successful `Accepted password for samm from 192.168.56.102` login, the corresponding session-opened event, and the connection being closed cleanly — the baseline "normal" SSH activity pattern.*

This confirms two things: `kali-attacker` can successfully authenticate to `soc-server` over SSH, and that successful authentication is correctly captured in Splunk with the correct source IP — the same field the brute-force detection logic depends on.

## Section 4 – SSH Brute Force Simulation

With a working baseline confirmed, repeated failed SSH login attempts were generated from `kali-attacker` against `soc-server` to simulate brute-force activity.

**Search used to review the failed attempts:**

```spl
index=main source="/var/log/auth.log" "Failed password"
```

**Result:** 5 failed password events over the test window, all against user `samm`, all from `192.168.56.102` (`kali-attacker`):

```
sshd-session[24431]: Failed password for samm from 192.168.56.102 port 38068 ssh2
sshd-session[24391]: Failed password for samm from 192.168.56.102 port 34040 ssh2
```

![Failed SSH password attempts from Kali](../../images/phase-05/11-ssh-bruteforce-failed-password-events.png)
*Screenshot placement: Splunk search results showing 5 `Failed password` events generated from `192.168.56.102` during the simulated brute-force window.*

**Aggregated detection query (the improved version of the Phase 4 detection logic, generalized to any source IP rather than a single hardcoded address):**

```spl
index=main source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort - count
```

**Result:**

| src_ip | count |
|---|---|
| 192.168.56.102 | 5 |

![Failed-login count grouped by source IP](../../images/phase-05/12-ssh-bruteforce-stats-count-by-srcip.png)
*Screenshot placement: the `stats count by src_ip` query correctly aggregating all 5 failed attempts under `192.168.56.102`, proving the field-extraction and aggregation logic identifies the attacking host on its own, without needing to know the IP in advance.*

**What this proves:** The `rex`/`stats` detection logic correctly and automatically identifies `kali-attacker` as the source generating repeated failed logins, using the exact same query structure introduced in Phase 4 — but now validated against real, externally generated brute-force traffic instead of a single test IP. This confirms the underlying search logic works correctly on its own.

**However — checking the Triggered Alerts page for this activity did not show a new alert instance.** This is investigated in full under Section 7, Problem 1, since it points to a real gap between "the detection query works" and "the saved scheduled alert actually fires for new attackers."

## Section 5 – Nmap Reconnaissance

The `nmap -sV` and `nmap -A` scans described in Section 2 were run from `kali-attacker` against `soc-server`'s SSH service as part of the same reconnaissance activity, and in doing so, generated authentication-log entries of their own — even though no login was ever attempted.

**Observed log entries in `auth.log` / Splunk, correlating with the scan window:**

```
sshd-session[29511]: Unable to negotiate with 192.168.56.102 port 57276: no matching key exchange method found.
Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1,diffie-hellman-group14-sha256,
diffie-hellman-group16-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group-exchange-sha256 [preauth]
```

![SSH pre-authentication negotiation failures from the Nmap scan](../../images/phase-05/09-nmap-preauth-negotiation-events.png)
*Screenshot placement: repeated `Unable to negotiate ... [preauth]` events in Splunk, each tied to a separate `sshd-session` ID, generated during the Nmap service-detection scan against `soc-server`'s SSH port.*

**Why this happened:** Nmap's service/version detection probe (`-sV`/`-A`) connects to the SSH port and offers a fixed, generic set of key-exchange algorithms as part of fingerprinting the service — including older algorithms like `diffie-hellman-group1-sha1` that this server's OpenSSH version (10.2p1) no longer supports by default for security reasons. The two sides can't agree on a key-exchange method, so the SSH connection is torn down during the negotiation phase itself.

**Why this is not a successful login — or a login attempt at all:** The `[preauth]` tag is the key detail. It means the connection failed *before* the authentication phase of the SSH protocol was ever reached — no username or password was submitted, and no credentials were tested. This is purely a transport-layer/protocol-negotiation failure triggered by how Nmap's probe is built, not any kind of authentication activity.

**Why this is still useful during an investigation:** Even though no login was attempted, a cluster of `[preauth]` negotiation failures from a single source IP, all within a few seconds of each other, is a recognizable artifact of a port/service scan against SSH. In a real investigation, this pattern — several fast, ephemeral SSH connections that all fail at the negotiation stage from the same host — is a useful supporting data point for identifying that a host was recently scanned, even if the scanning tool itself never authenticated or is no longer active.

> **Important distinction:** This is **not** an IDS detection. Splunk did not flag, score, or alert on these events in this phase — they were manually identified by searching `auth.log` after the fact. The value here is that the *raw log data* is available for an analyst to find during an investigation, not that any automated detection currently exists for this pattern.

## Section 6 – Verification

| What Was Verified | How |
|---|---|
| SSH connectivity between `kali-attacker` and `soc-server` | Confirmed via the successful `Accepted password` event in Section 3, and via Nmap reporting port 22 open with a fingerprinted SSH banner in Section 2 |
| Splunk log ingestion from the new activity | Confirmed by running `index=main source="/var/log/auth.log"` searches and seeing new events appear with the correct `192.168.56.102` source IP, matching timestamps to when the activity was actually performed |
| Alert triggering | Checked directly on the Triggered Alerts page — result documented honestly in Section 7, Problem 1: no new alert instance appeared for this phase's activity |
| Authentication events | Confirmed via the baseline (`Accepted password`), brute-force (`Failed password`), and unrelated (`sudo`) searches, each returning the expected event types and fields |
| Nmap-related SSH events | Confirmed by correlating the `[preauth]` negotiation-failure timestamps in Splunk against the time the Nmap scan was actually run on `kali-attacker` |

## Section 7 – Problems Encountered

### Problem 1: Brute-force simulation did not produce a new Triggered Alert

**Symptoms:** After generating 5 failed SSH login attempts from `kali-attacker` (`192.168.56.102`) and confirming via the `stats count by src_ip` query that the detection logic correctly identified the activity, the **Triggered Alerts** page still showed only the original alert instance from `2026-07-27 08:15:01 UTC` — no new entry for this phase's activity appeared.

**Root Cause:** The saved, scheduled alert (`SSH Brute Force Detection`, configured in Phase 4) uses the original test query, which filters explicitly to a hardcoded source IP:
```spl
index=main source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| search src_ip="10.0.2.2"
```
This query only ever matches failed attempts from `10.0.2.2` — the IP used during Phase 4 testing. Since `kali-attacker`'s address (`192.168.56.102`) does not match that hardcoded filter, the saved alert has no way to fire for this phase's brute-force activity, regardless of how many failed attempts occurred.

![Triggered Alerts page still showing only the original instance](../../images/phase-05/15-triggered-alerts-unchanged.png)
*Screenshot placement: the Triggered Alerts page showing only the single `2026-07-27 08:15:01 UTC` entry — no new trigger corresponding to the Aug 5 activity from `kali-attacker`.*

![The saved alert's underlying query, scoped to the original test IP](../../images/phase-05/16-original-alert-search-hardcoded-ip.png)
*Screenshot placement: the original saved search behind the alert, confirming the hardcoded `search src_ip="10.0.2.2"` filter that explains why it did not fire for the new attacker IP.*

**Solution:** Not yet applied to the saved alert. The manually-run `stats count by src_ip` query (Section 4) already demonstrates the correct, generalized replacement logic — the saved alert itself still needs to be updated to use that version instead of the hardcoded IP filter. This is being tracked as a follow-up rather than resolved in this phase, since applying it live would change previously-documented Phase 4 configuration.

**Verification:** Confirmed by directly comparing the alert's saved search definition against the source IP actually present in this phase's `Failed password` events.

**Lesson Learned:** A detection query that was correct for its original test scenario can silently stop being useful the moment the actual threat comes from a different source than the one it was validated against. Hardcoding a specific value (like a test IP) during development is reasonable, but a saved, scheduled production alert needs to be revisited and generalized — otherwise "the alert works" and "the alert would catch a real attacker" can quietly become two different claims.

---

### Problem 2: Accidental sudo authentication failure during unrelated administrative work

**Symptoms:** While running routine administrative commands on `soc-server`, `sudo apt update` failed twice in a row with `sudo: Authentication failed, try again.`, followed by `sudo: maximum 3 incorrect authentication attempts`, temporarily locking out further `sudo` attempts in that terminal session.

**Root Cause:** The password was mistyped twice in a row when prompted by `sudo`, exhausting the default retry limit of three attempts.

![Terminal showing repeated sudo authentication failures](../../images/phase-05/13-sudo-authentication-failure-terminal.png)
*Screenshot placement: terminal on `soc-server` showing `sudo apt update` failing twice with `Authentication failed, try again.`, followed by the `maximum 3 incorrect authentication attempts` lockout message.*

**Solution:** Re-ran the command in a fresh attempt and entered the correct password, which succeeded without further issue.

**Verification:** Confirmed the event was captured correctly in Splunk, which recorded both the failed authentication attempt and the subsequent activity:
```
sudo: pam_unix(sudo:auth): authentication failure; logname=samm uid=1000 euid=0 tty=/dev/pts/0 ruser=samm rhost= user=samm
sudo: pam_unix(sudo:session): session opened for root by samm
sudo: samm : TTY=/dev/tty1 ; PWD=/home/samm ; USER=root ; COMMAND=/usr/bin/su - splunk
```

![Splunk capturing the sudo authentication failure](../../images/phase-05/14-sudo-authentication-failure-splunk.png)
*Screenshot placement: Splunk search `index=main source="/var/log/auth.log" sudo` showing the authentication-failure event alongside subsequent successful `sudo` activity, confirming both the mistake and the recovery were logged accurately.*

**Lesson Learned:** This wasn't an attack, but it's a useful reminder that the same logging pipeline built to catch malicious activity also faithfully captures ordinary operator mistakes — which is exactly the intended behavior. It's also a small real-world reminder of why account lockout thresholds exist: even a legitimate, authorized user can trip one accidentally.

## Section 8 – Windows Log Collection (Work in Progress)

Universal Forwarder installation on `win-client` has been started, but Windows log integration into Splunk is still under investigation. This section will be fully documented — including configuration steps, verification, and any issues encountered — once that work is complete.

## References

- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [OpenSSH `sshd_config` — key exchange algorithm documentation](https://man.openbsd.org/sshd_config)
- [Splunk `stats` Command Documentation](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/Stats)

## Next Phase

➡️ Phase 6 — Incident Response *(planned)*: using the multi-host activity captured in this phase (reconnaissance, baseline login, and brute-force simulation) as the basis for a documented investigation and response workflow, and generalizing the SSH Brute Force Detection alert identified as a gap in Section 7.
