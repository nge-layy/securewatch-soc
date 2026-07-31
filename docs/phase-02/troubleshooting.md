# Phase 2 - Problems Encountered & Solutions

## Overview

During this phase, I encountered several issues while configuring SSH remote access. Troubleshooting these problems helped me better understand how SSH authentication, VirtualBox networking, and Linux services work together.

---

## Problem 1 — SSH Authentication Failed

### Symptom

When connecting from Windows, SSH returned:

```text
sam@localhost's password:
Permission denied, please try again.
```

### Root Cause

The SSH server was running correctly, but the username or password entered during login was incorrect.

### Solution

I verified:

- The correct Linux username.
- The correct password.
- That the account existed on the Ubuntu Server.

After confirming the credentials, I connected successfully.

### Lesson Learned

Receiving **"Permission denied"** usually means the network connection is working correctly. The problem is authentication rather than networking or the SSH service itself.

---

## Problem 2 — Unable to Connect Through SSH

### Symptom

The Windows computer could not establish an SSH connection to the Ubuntu Server.

### Root Cause

The VirtualBox NAT port forwarding rule had not been configured correctly.

### Solution

I opened the VirtualBox network settings and created the following rule.

| Setting | Value |
|----------|-------|
| Protocol | TCP |
| Host IP | 127.0.0.1 |
| Host Port | 2222 |
| Guest Port | 22 |

After applying the rule, I connected using:

```powershell
ssh -p 2222 sam@127.0.0.1
```

The connection was established successfully.

### Lesson Learned

Before troubleshooting SSH itself, always verify that the network path exists. If port forwarding is incorrect, the SSH server will never receive the connection request.

---

## Problem 3 — Verifying the SSH Service

### Symptom

Before attempting remote login, I needed to confirm that the SSH service was actually running.

### Solution

I checked the service status with:

```bash
sudo systemctl status ssh
```

I also verified that the service would automatically start after every reboot.

```bash
sudo systemctl enable ssh
```

### Lesson Learned

Always verify the service locally before troubleshooting from another computer. This helps separate service issues from network issues.

---

# Summary Table

| Problem | Root Cause | Solution |
|----------|------------|----------|
| Permission denied | Incorrect login credentials | Verified username and password |
| Unable to connect | NAT port forwarding not configured correctly | Created the VirtualBox port forwarding rule |
| Needed to verify SSH | Service status unknown | Checked and enabled the SSH service |

---

# What I Learned

This phase taught me that troubleshooting SSH should follow a logical order.

1. Verify that the SSH service is running.
2. Confirm the network configuration and port forwarding.
3. Test the connection.
4. Check authentication if the connection reaches the server.

Following these steps makes it much easier to identify where a problem occurs instead of guessing.
