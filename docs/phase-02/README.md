# Phase 2 - Remote Administration

## Overview

In this phase, I configured secure remote access to my Ubuntu Server using SSH. This allows me to manage the server from my Windows computer without opening the VirtualBox console.

SSH is one of the most common tools used by Linux administrators and SOC analysts to securely manage remote systems.

---

## Objectives

During this phase, I completed the following tasks:

- Installed the OpenSSH Server.
- Started and enabled the SSH service.
- Configured VirtualBox NAT port forwarding.
- Connected to `soc-server` from my Windows computer using SSH.
- Verified that remote administration was working correctly.

---

## Tools Used

The following tools were used during this phase:

- **OpenSSH Server** – Provides secure remote access to the Ubuntu Server.
- **systemd** – Used to start, stop, and manage the SSH service.
- **VirtualBox NAT Port Forwarding** – Allows the Windows host to connect to the virtual machine.
- **Windows SSH Client** – Used to connect to the Ubuntu Server from Windows.

---

## What is SSH?

SSH (Secure Shell) is a secure protocol used to connect to another computer over a network. It encrypts the connection so that usernames, passwords, and commands are protected while they travel between the client and the server.

SSH is the standard method for managing Linux servers remotely.

---

## Why This Matters

Most Linux servers are managed remotely instead of through a monitor and keyboard. Learning how to install, configure, and troubleshoot SSH is an important skill for Linux administrators and SOC analysts.

---

## Architecture

The SSH connection and VirtualBox port forwarding used in this phase are shown in [architecture.md](architecture.md).

---

## What I Did

During this phase, I:

1. Installed the OpenSSH Server.
2. Started the SSH service and enabled it to start automatically after reboot.
3. Configured a NAT port forwarding rule in VirtualBox.
4. Connected to `soc-server` from my Windows computer using SSH.
5. Confirmed that the remote session worked correctly.

For the commands used in this phase, see [commands.md](commands.md).

If you encounter similar issues, see [troubleshooting.md](troubleshooting.md).

---

## Security Considerations

During this phase, I followed these basic security practices:

- Only the SSH service was exposed for remote administration.
- Only one port was forwarded in VirtualBox.
- Password authentication was used during the initial setup. SSH key authentication will be added in a later phase.

---

## What I Learned

This phase helped me understand that:

- A service must be started before it can be used.
- Enabling a service allows it to start automatically after a reboot.
- Port forwarding should be limited to only the services that are needed.
- Keeping one SSH session open while testing changes can help prevent accidental lockouts.

---

## Verification

Before moving to the next phase, I confirmed that:

- OpenSSH Server was installed.
- The SSH service was running.
- The SSH service started automatically after reboot.
- VirtualBox NAT port forwarding was configured correctly.
- I successfully connected to `soc-server` from my Windows computer.

---

## References

- OpenSSH Documentation
- Oracle VirtualBox User Manual

---

## Next Phase

**Phase 3 - Logging & SIEM Preparation**

In the next phase, I will configure system logging and prepare the server for Splunk integration so security events can be collected and analyzed.
