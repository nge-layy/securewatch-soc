# Phase 3 — Problems Encountered & Solutions

---

### Problem 1: `splunk` user couldn't read `/var/log/audit/audit.log` despite being in `adm`

**Symptom:**
```
sudo -u splunk cat /var/log/audit/audit.log
cat: /var/log/audit/audit.log: Permission denied
```
Meanwhile, `auth.log` and `syslog` were readable without issue.

**Root Cause:** Unlike most logs under `/var/log/`, `auditd` by default restricts `audit.log` to root-only access (`0600`) rather than group-readable, specifically because audit logs are considered especially sensitive and tamper-relevant.

**Solution:** Adjusted the audit log's group ownership and permission mode to allow group read access, then confirmed this was consistent with the `adm`-based access model used for the rest of the log set:
```bash
sudo chgrp adm /var/log/audit/audit.log
sudo chmod 640 /var/log/audit/audit.log
```
For persistence across log rotation, the equivalent setting was also applied in `/etc/audit/auditd.conf` (`log_group = adm`).

**Lesson:** Not every log source follows the same default permission convention — `auditd` deliberately locks its log down harder than syslog-based logs by default, which makes sense given its role, but means the "just add to `adm`" approach needed an explicit follow-up step here rather than working automatically.

---

### Problem 2: `journalctl` returned "No journal files were found" when queried as the `splunk` user

**Symptom:**
```
sudo -u splunk journalctl -u ssh.service
No journal files were opened due to insufficient permissions.
```

**Root Cause:** Reading the systemd journal requires membership in the `systemd-journal` group specifically — `adm` group membership alone does not grant journal read access, since `journald` maintains its own separate access group.

**Solution:**
```bash
sudo usermod -aG systemd-journal splunk
```

**Lesson:** Ubuntu's logging stack isn't a single unified permission system — `adm` covers traditional syslog-based text logs, while `journald` has its own group. Assuming one group covers "all logs" is an easy and understandable mistake; verifying access per log *source*, not just per log *directory*, caught this gap.

---

### Problem 3: Audit rule didn't survive a reboot

**Symptom:** A rule added with `auditctl -w /etc/passwd -p wa -k passwd_changes` worked immediately, but `ausearch -k passwd_changes` returned nothing for events generated after a VM restart.

**Root Cause:** `auditctl` rules added directly at the command line are runtime-only and are not persisted to disk — they're wiped on reboot unless also written into the rules directory.

**Solution:** Added the rule to `/etc/audit/rules.d/audit.rules` and reloaded with `augenrules --load`, as documented in [commands.md](commands.md), so the rule is reapplied automatically on every boot.

**Lesson:** This mirrors the exact "enable vs. start" distinction from Phase 2 (SSH) — testing that something *works right now* is not the same as confirming it will *still work after a restart*. Every configuration change made in this project is now checked against both conditions.

---

## Summary Table

| Problem | Root Cause | Fix | Prevention |
|---|---|---|---|
| `splunk` couldn't read `audit.log` | `auditd` defaults to root-only (`0600`) log permissions | `chgrp adm` + `chmod 640` on the log, plus `auditd.conf` setting | Verify permissions per log file, don't assume `adm` covers everything |
| `journalctl` denied for `splunk` | `journald` uses its own `systemd-journal` group | Add `splunk` to `systemd-journal` group | Check access per log *subsystem*, not just per directory |
| Audit rule lost after reboot | Runtime-only `auditctl` rule not persisted | Add rule to `/etc/audit/rules.d/`, reload with `augenrules` | Always persist configuration changes, then verify post-reboot |
