# Phase 1 - Infrastructure Foundation

## Overview

In this phase, I built the foundation for my SecureWatch Home SOC project.

I installed Oracle VirtualBox, created an Ubuntu Server virtual machine, configured networking, and prepared the system for the later phases of the project. This phase focuses on creating a stable and secure environment before installing security tools.

---

## Objectives

During this phase, I completed the following tasks:

- Installed Oracle VirtualBox on my Windows 11 computer.
- Created an Ubuntu Server virtual machine named `soc-server`.
- Configured NAT networking and verified internet connectivity.
- Updated the operating system and checked basic system information.
- Created the project folder structure that will be used throughout the SecureWatch Home SOC project.

For the commands used in this phase, see [commands.md](commands.md).

If you encounter similar issues, see [troubleshooting.md](troubleshooting.md).

---

## Technologies Used

The following tools were used during this phase:

- **Oracle VirtualBox** – Used to create and run the Ubuntu Server virtual machine.
- **Ubuntu Server LTS** – The operating system running inside the virtual machine.
- **VirtualBox NAT Networking** – Provides internet access while keeping the virtual machine isolated from the home network.
- **APT** – Ubuntu's package manager for installing software and system updates.

---

## Why I Used a Virtual Machine

I chose to use a virtual machine because it provides a safe environment for learning and testing.

If I make a mistake, I can restore a snapshot instead of reinstalling the operating system. Using a virtual machine also keeps my lab separate from my main computer, which is useful because later phases will include security monitoring and attack simulations.

---

## Architecture

The architecture and network design for this phase are documented in [architecture.md](architecture.md).

---

## Security Considerations

During this phase, I followed these basic security practices:

- Used **NAT networking** to keep the virtual machine isolated from my home network.
- Updated the operating system before installing additional software.
- Used **Ubuntu Server** instead of Ubuntu Desktop to reduce unnecessary software and the overall attack surface.

---

## What I Learned

This phase helped me understand the importance of building a solid foundation before adding security tools.

I learned that:

- Checking network connectivity first makes troubleshooting much easier.
- Recording basic system information provides a useful baseline for future comparisons.
- NAT networking offers internet access while keeping the lab isolated from the home network.

---

## Verification

Before moving to the next phase, I confirmed that:

- Oracle VirtualBox was installed and working.
- The Ubuntu Server virtual machine booted successfully.
- The hostname was correctly set to `soc-server`.
- The server had internet access.
- The operating system was fully updated.
- The project folder structure was created successfully.

---

## References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Oracle VirtualBox Networking Modes](https://www.virtualbox.org/manual/ch06.html)

---

## Next Phase

**Phase 2 — Remote Administration**

In the next phase, I will install and configure SSH so the Ubuntu Server can be managed remotely from my Windows host.
