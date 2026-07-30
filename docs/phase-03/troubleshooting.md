# Phase 3 - Problems Encountered & Solutions

## Problem 1: auditd service not found

**Problem**

After trying to check the audit service, Ubuntu reported that the service could not be found.

**Cause**

The `auditd` package had not been installed.

**Solution**

```bash
sudo apt install auditd audispd-plugins -y
```

After installation, I started and enabled the service.

**Lesson**

Always confirm that a package is installed before troubleshooting the service.

---

## Problem 2: Splunk couldn't read log files

**Problem**

The Splunk service account couldn't access the required system logs.

**Cause**

The `splunk` user was not a member of the `adm` group.

**Solution**

```bash
sudo usermod -aG adm splunk
```

Then I confirmed the group membership.

```bash
groups splunk
```

**Lesson**

Linux permissions depend on user and group membership.

---

## Problem 3: Splunk disk space warning

**Problem**

Splunk displayed a warning about low disk space in the dispatch directory.

**Cause**

My virtual machine only had a small virtual disk.

**Solution**

I cleaned unnecessary files and monitored disk usage.

```bash
df -h
du -sh /opt/splunk/*
```

**Lesson**

Always monitor available storage when running SIEM software.
