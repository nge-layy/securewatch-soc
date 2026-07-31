# Phase 2 - Architecture & Connection Design

## Overview

In Phase 1, the Ubuntu Server was connected to the internet using VirtualBox NAT networking. While this allowed the server to download updates, it also meant the Windows host could not connect directly to the virtual machine.

To solve this, I configured **VirtualBox NAT Port Forwarding**. This allows only the SSH service to be accessed from my Windows computer while keeping the Ubuntu Server isolated from the rest of my home network.

This design follows the principle of exposing only the services that are required.

---

# Remote Administration Architecture

```text
                     Windows 11 Host
               +-------------------------+
               | SSH Client (OpenSSH)    |
               |                         |
               | ssh -p 2222 sam@127.0.0.1
               +------------+------------+
                            |
                            |
                     localhost:2222
                            |
                            ▼
                VirtualBox NAT Engine
            Port Forward: 2222 → 22
                            |
                            ▼
              Ubuntu Server (soc-server)
              SSH Server (sshd - Port 22)
                            |
                            ▼
               Remote Terminal Session
```

---

# Port Forwarding Configuration

The following NAT rule was configured inside VirtualBox.

| Setting | Value |
|----------|-------|
| Name | SSH |
| Protocol | TCP |
| Host IP | 127.0.0.1 |
| Host Port | 2222 |
| Guest Port | 22 |

The Host IP was set to **127.0.0.1**, which means only the Windows host can access the forwarded port. Other devices on the home network cannot connect to the Ubuntu Server through this rule.

---

# Why I Used Port Forwarding

Because the virtual machine uses NAT networking, it is hidden behind the Windows host.

Without port forwarding:

- Windows cannot directly reach the Ubuntu Server.
- SSH connections will fail.

By forwarding port **2222** on the Windows host to port **22** on the Ubuntu Server, Windows can communicate with the SSH server while the VM remains isolated from the rest of the network.

This provides a good balance between security and usability for a home lab.

---

# SSH Connection Process

The remote connection follows these steps:

1. I open PowerShell on Windows.
2. I run the SSH command.

```powershell
ssh -p 2222 sam@127.0.0.1
```

3. The SSH client connects to **localhost** on port **2222**.
4. VirtualBox forwards the connection to port **22** inside the Ubuntu Server.
5. The SSH server (`sshd`) receives the connection.
6. After successful authentication, an encrypted terminal session is created.

---

# Host Key Verification

The first time I connected to `soc-server`, SSH displayed the server's fingerprint and asked whether I trusted the server.

After I accepted it, Windows saved the fingerprint in the **known_hosts** file.

On future connections, SSH compares the stored fingerprint with the server's current fingerprint.

If the fingerprint changes unexpectedly, SSH displays a warning because the server may have been reinstalled or someone may be attempting a man-in-the-middle attack.

---

# SSH Service Startup

The SSH service is managed by **systemd**.

```text
System Boot
      │
      ▼
systemd
      │
      ▼
Is SSH Enabled?
      │
 ┌────┴─────┐
 │          │
Yes        No
 │          │
 ▼          ▼
SSH Starts  Manual Start Required
Automatically
```

Running

```bash
sudo systemctl enable ssh
```

configures SSH to start automatically whenever the server boots.

Running

```bash
sudo systemctl start ssh
```

starts the SSH service immediately without waiting for the next reboot.

Both commands were used to make sure the server is always available for remote administration.

---

# Security Considerations

Several security decisions were made during this phase.

- Only the SSH service is exposed through port forwarding.
- The forwarded port is bound to **127.0.0.1**, preventing other devices on the home network from connecting.
- NAT networking continues to isolate the Ubuntu Server from the LAN.
- Password authentication was used initially to verify connectivity. SSH key authentication will be configured in a later hardening phase.

---

# Summary

This architecture allows secure remote administration of `soc-server` without exposing the entire virtual machine to the home network.

Using VirtualBox NAT together with port forwarding provides a simple and secure design that mirrors how administrators often access servers through controlled entry points while keeping unnecessary services hidden.
