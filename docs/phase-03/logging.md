# Phase 3 - Understanding Linux Logs

Before integrating Splunk, I explored the main log files available on Ubuntu Server.

## Common Log Files

| Log File | What It Records |
|----------|-----------------|
| `/var/log/auth.log` | SSH logins, `sudo` commands, and authentication events |
| `/var/log/syslog` | General system and application messages |
| `journalctl` | Logs managed by systemd |
| `/var/log/audit/audit.log` | Security events recorded by `auditd` |

---

## Commands I Used

View authentication logs:

```bash
sudo tail -f /var/log/auth.log
```

View system logs:

```bash
sudo tail -f /var/log/syslog
```

View systemd logs:

```bash
journalctl
```

View audit logs:

```bash
sudo cat /var/log/audit/audit.log
```

---

## What I Learned

- Different Linux logs record different types of events.
- `auth.log` is useful for monitoring SSH logins and `sudo` activity.
- `syslog` contains general system messages.
- `journalctl` displays logs managed by systemd.
- `auditd` records additional security events that can help during investigations.

These logs will be collected by Splunk in the next phases of this project.
