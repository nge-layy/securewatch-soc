# Phase 2 - Command Reference

---

These are the main commands I used to configure and verify SSH remote access.

---

## 1. Install OpenSSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
```

**Purpose**

Install the OpenSSH Server package and make sure the package list is up to date.

---

## 2. Check the SSH Service

```bash
sudo systemctl status ssh
```

**Purpose**

Verify that the SSH service is running correctly.

---

## 3. Enable SSH at Startup

```bash
sudo systemctl enable ssh
```

**Purpose**

Configure SSH to start automatically whenever the server boots.

---

## 4. Start or Restart SSH

```bash
sudo systemctl start ssh
sudo systemctl restart ssh
```

**Purpose**

Start the SSH service or restart it after making configuration changes.

---

## 5. Verify SSH is Listening

```bash
sudo ss -tulpn | grep ssh
```

**Purpose**

Confirm that SSH is listening on port **22** and ready to accept connections.

---

## 6. Check the Server IP Address

```bash
ip a
```

**Purpose**

View the server's IP address for verification and troubleshooting.

---

## 7. Configure VirtualBox Port Forwarding

**VirtualBox → Settings → Network → Adapter 1 → Advanced → Port Forwarding**

| Setting | Value |
|---------|-------|
| Name | SSH |
| Protocol | TCP |
| Host IP | 127.0.0.1 |
| Host Port | 2222 |
| Guest Port | 22 |

**Purpose**

Forward port **2222** on the Windows host to port **22** on the Ubuntu Server so SSH can connect through NAT.

---

## 8. Connect from Windows

```powershell
ssh -p 2222 username@127.0.0.1
```

**Purpose**

Connect to the Ubuntu Server from the Windows computer using SSH.

The first connection will ask you to confirm the server's fingerprint. After accepting it, future connections will remember the server unless its fingerprint changes.

---

## 9. Verify the Connection

```bash
whoami
hostname
```

**Purpose**

Confirm that you are connected to the correct user account and server.
