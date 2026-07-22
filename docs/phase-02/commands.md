# Phase 2 — Command Reference

---

### 1. Install the OpenSSH server

```bash
sudo apt update
sudo apt install openssh-server -y
```

**Why:** Ubuntu Server does not install an SSH daemon by default on every image, so this ensures it's present. Running `apt update` first guarantees the installed version pulls from a current package index rather than a stale one.

---

### 2. Check the SSH service status

```bash
sudo systemctl status ssh
```

**Why:** Confirms the daemon is actually running before attempting a remote connection — troubleshooting a "connection refused" error is much faster when you've already verified the service state locally.

**Expected output (excerpt):**
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

The `enabled` flag in the `Loaded` line confirms it will also start automatically on boot.

---

### 3. Enable SSH to start on boot (if not already enabled)

```bash
sudo systemctl enable ssh
```

**Why:** Without this, `sshd` would need to be started manually after every reboot — unacceptable for a server meant to be managed remotely, since a reboot would otherwise require console access just to bring SSH back online.

---

### 4. Start (or restart) the SSH service

```bash
sudo systemctl start ssh
sudo systemctl restart ssh   # used after config changes
```

**Why:** `start` is used the first time the service is brought up; `restart` is used any time `/etc/ssh/sshd_config` is modified, since `sshd` does not hot-reload configuration changes automatically.

---

### 5. Confirm sshd is listening on the expected port

```bash
sudo ss -tulpn | grep ssh
```

**Why:** Verifies at the network-socket level that `sshd` is actually bound and listening on port 22, independent of what `systemctl status` reports. This catches edge cases where the service reports "active" but failed to bind to the port (e.g., a port conflict).

**Expected output:**
```
tcp   LISTEN  0  128  0.0.0.0:22   0.0.0.0:*   users:(("sshd",pid=...,fd=3))
```

---

### 6. Identify the guest's NAT IP (for reference/logging, not for direct host access)

```bash
ip a | grep inet
```

**Why:** Even though the host reaches the guest through the forwarded port rather than the guest's IP directly, recording the guest's internal address is useful for troubleshooting and for cross-referencing later against SSH and auth logs.

---

### 7. Configure VirtualBox NAT port forwarding

*(Performed in the VirtualBox Manager GUI, not the guest shell)*

```
VM Settings → Network → Adapter 1 → Advanced → Port Forwarding
  Name:      SSH
  Protocol:  TCP
  Host IP:   127.0.0.1
  Host Port: 2222
  Guest IP:  (blank — VirtualBox routes internally)
  Guest Port: 22
```

**Why:** This is the rule that actually makes the guest reachable from the host. Binding **Host IP** to `127.0.0.1` (rather than leaving it blank, which defaults to all interfaces) ensures only the host machine itself — not other devices on the home network — can use this forwarded port.

---

### 8. Connect from the Windows host

```powershell
ssh -p 2222 username@127.0.0.1
```

**Why:** Connects to the loopback address on the forwarded port, which VirtualBox's NAT engine transparently routes to port 22 inside the guest. The `-p` flag is required here because SSH defaults to port 22, but the host-side forwarded port is 2222 in this setup.

**Expected output (first connection):**
```
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Typing `yes` adds the host key to `known_hosts`, so future connections won't repeat this prompt unless the server's host key changes — which would indicate either a reinstalled server or a potential man-in-the-middle condition worth investigating.

---

### 9. Verify the remote session

```bash
whoami
hostname
```

**Why:** A simple sanity check once connected — confirms the session landed on the correct user and correct machine, which matters once multiple lab VMs or jump hosts are in play.
