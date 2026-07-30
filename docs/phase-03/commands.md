# Phase 3 - Command Reference

These are the main commands I used to prepare the server for log collection and Splunk integration.

---

## 1. Install auditd

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
```

**Purpose**

Install the Linux Audit Daemon.

---

## 2. Start and Enable auditd

```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

**Purpose**

Start the audit service and make sure it starts automatically after reboot.

**Verify**

```bash
sudo systemctl status auditd
```

---

## 3. View Authentication Logs

```bash
sudo tail -f /var/log/auth.log
```

**Purpose**

Monitor authentication events such as SSH logins and `sudo` activity.

---

## 4. View System Logs

```bash
sudo tail -f /var/log/syslog
```

**Purpose**

Monitor general system messages.

---

## 5. View Systemd Logs

```bash
journalctl
```

**Purpose**

View logs managed by systemd.

---

## 6. Verify the splunk User

```bash
id splunk
```

**Purpose**

Confirm that the Splunk service account exists.

---

## 7. Add splunk to the adm Group

```bash
sudo usermod -aG adm splunk
```

**Purpose**

Allow the Splunk service account to read system log files.

**Verify**

```bash
groups splunk
```

---

## 8. Test Log Access

```bash
sudo -u splunk head -5 /var/log/auth.log
```

**Purpose**

Verify that the `splunk` account can read authentication logs.
