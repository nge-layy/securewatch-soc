# Phase 3 — Command Reference

---

### 1. Install auditd

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
```

**Why:** `auditd` is not installed by default on a minimal Ubuntu Server image. `audispd-plugins` extends `auditd` with plugins for dispatching audit events to external tools — useful groundwork for eventual SIEM integration.

---

### 2. Enable and start auditd

```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

**Why:** As with SSH in Phase 2, enabling ensures auditing survives a reboot — a gap in audit coverage after a restart is exactly the kind of blind spot a SOC can't afford.

**Verification:**
```bash
sudo systemctl status auditd
```
```
● auditd.service - Security Auditing Service
     Loaded: loaded (/lib/systemd/system/auditd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

---

### 3. Add an audit rule (example: monitor `/etc/passwd` for changes)

```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

**Why:** `/etc/passwd` defines every user account on the system — unauthorized modification is a high-severity indicator. This rule specifically watches for write and attribute-change events, tagging matches with a searchable key.

---

### 4. Make audit rules persistent across reboots

```bash
echo "-w /etc/passwd -p wa -k passwd_changes" | sudo tee -a /etc/audit/rules.d/audit.rules
sudo augenrules --load
```

**Why:** Rules added with `auditctl` directly are only active until the next reboot. Writing them into `/etc/audit/rules.d/audit.rules` and reloading with `augenrules` ensures the rule persists — critical, since a rule that silently disappears after a restart creates an audit gap nobody notices until it's needed.

---

### 5. Query audit events by key

```bash
sudo ausearch -k passwd_changes
```

**Why:** `ausearch` filters the (often large and dense) audit log down to just the events matching a specific rule key, making investigation practical instead of manually parsing raw `audit.log`.

---

### 6. Create the `splunk` service account

```bash
sudo useradd -r -s /usr/sbin/nologin splunk
```

**Why:** `-r` creates a **system account** (appropriate for a service, not a human user) with a UID in the reserved system range. `-s /usr/sbin/nologin` explicitly disables interactive shell login for this account — it should only ever run the Splunk forwarder process, never be used to log in and get a shell. This is least privilege applied directly: the account exists for one function and is technically incapable of being used for anything else.

**Verification:**
```bash
id splunk
```
```
uid=999(splunk) gid=999(splunk) groups=999(splunk)
```

---

### 7. Add the `splunk` account to the `adm` group

```bash
sudo usermod -aG adm splunk
```

**Why:** Grants read access to the system logs the `adm` group traditionally covers (`/var/log/syslog`, `/var/log/auth.log`, and others), without granting `sudo` or any administrative capability. As with Phase 2.5's `usermod -aG`, the `-a` flag is essential to avoid stripping the account's existing group memberships.

**Verification:**
```bash
groups splunk
```
```
splunk : splunk adm
```

---

### 8. Verify log read access from the splunk account's perspective

```bash
sudo -u splunk cat /var/log/auth.log | tail -5
```

**Why:** Running the read as the `splunk` user (rather than as root or the admin account) is the only way to actually confirm the permission grant works as intended from that account's real-world perspective — testing as root would succeed regardless of whether `splunk`'s own permissions were correct.

**Expected output:** The tail end of `auth.log`, confirming successful read access.

---

### 9. Confirm the splunk account cannot write to logs

```bash
sudo -u splunk touch /var/log/auth.log.test
```

**Why:** A deliberate negative test. This command *should* fail with a permission error — confirming the account has read-only access, not read-write, which protects log integrity.

**Expected output:**
```
touch: cannot touch '/var/log/auth.log.test': Permission denied
```

---

### 10. Confirm the splunk account cannot use sudo

```bash
sudo -u splunk sudo whoami
```

**Why:** Another deliberate negative test, confirming the account has no path to privilege escalation.

**Expected output:**
```
splunk is not in the sudoers file. This incident will be reported.
```

This message confirms the account is correctly scoped — it can read logs and do nothing else.
