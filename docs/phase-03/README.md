# Phase 3 - Logging & SIEM Preparation

## Overview

In this phase, I prepared my Ubuntu Server for Security Information and Event Management (SIEM).

I explored how Linux generates and stores system logs, installed **auditd** for security auditing, and installed **Splunk Enterprise** to collect and analyze those logs.

I also configured the `splunk` service account with the minimum permissions required to read system logs, following the principle of least privilege.

This phase provides the logging foundation for the threat detection and incident response activities that will be developed in later phases.

---

# Objectives

During this phase, I completed the following tasks:

- Explored the default Linux log files.
- Learned how Linux stores system events.
- Installed and configured `auditd`.
- Installed Splunk Enterprise.
- Started the Splunk service.
- Verified the Splunk Web interface.
- Reviewed the permissions required for log collection.
- Verified that Splunk could access the required log files.

See **commands.md** for the complete command reference and **troubleshooting.md** for the issues encountered during this phase.

---

# Tools Used

| Tool | Purpose |
|------|---------|
| `auth.log` | Records authentication events such as SSH logins and sudo usage |
| `syslog` | Stores general operating system messages |
| `journalctl` | Displays logs managed by systemd |
| `auditd` | Records security-related events |
| Splunk Enterprise | Collects, indexes, and searches log data |
| Linux `adm` Group | Grants read access to system log files |

---

# Architecture

See **architecture.md** for the complete architecture and log flow diagrams.

---

# Linux Log Sources

Ubuntu stores different types of information in different log files.

| Log Source | Purpose |
|------------|---------|
| `auth.log` | SSH logins, authentication, sudo activity |
| `syslog` | General operating system messages |
| `journalctl` | Logs from systemd services |
| `auditd` | Security auditing and monitored events |

Understanding these log sources is important because a SIEM relies on them to detect suspicious activity.

---

# What I Did

During this phase, I:

- Reviewed the default Linux log files.
- Installed and enabled `auditd`.
- Installed Splunk Enterprise.
- Started the Splunk service.
- Accessed the Splunk Web interface.
- Verified the Splunk service was running.
- Granted the `splunk` account permission to read system log files.
- Confirmed that the required logs were accessible.

---

# Why Splunk Needs Log Access

Splunk analyzes information stored in log files.

Instead of giving the Splunk service full administrator privileges, I granted it only the permissions required to read the logs.

This follows the **Principle of Least Privilege**, which reduces the impact of a compromised service account.

---

# Security Considerations

During this phase, I followed several security practices.

- Installed Splunk using a dedicated service account.
- Avoided giving unnecessary administrator privileges.
- Granted only read access to required log files.
- Verified that logging services were working before configuring Splunk.
- Confirmed that important system logs were available for future monitoring.

---

# What I Learned

This phase helped me understand that:

- Different Linux logs record different types of events.
- `auditd` provides more detailed security auditing than standard system logs.
- Splunk depends on reliable log sources.
- Service accounts should only receive the permissions they actually need.
- Preparing good log sources is an important step before creating alerts and detections.

---

# Verification Checklist

Before moving to the next phase, I confirmed:

-  `auditd` installed and running
- System logs available
- Splunk Enterprise installed
- Splunk service running
- Splunk Web interface accessible
- Required log files readable
- Log collection environment ready for SIEM configuration

---

# References

- Linux Audit Framework Documentation
- Ubuntu Server Documentation
- Splunk Enterprise Documentation

---

# Next Phase

**Phase 4 – Threat Simulation & Log Collection**

The next phase focuses on generating security events, collecting them in Splunk, and preparing the environment for detection and analysis.
