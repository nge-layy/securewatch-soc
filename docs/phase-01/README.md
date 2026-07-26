# Phase 1 - Infrastructure Foundation

## Overview

Phase 1 is about setting up the machine I will defend. I create a virtual server, give it a clear network identity, and make sure it can connect to the internet for updates.  I build the server step by step, deciding how to divide the disk, choosing the network mode, and giving it a hostname. This helps me practice the same skills a junior system administrator or SOC analyst would use when building lab systems in real work.

## Objectives

 I completed the following tasks:
- Installed Oracle VirtualBox on my Windows 11 computer.
- Created an Ubuntu Server virtual machine named `soc-server`.
- Configured NAT networking and verified that the server could access the internet.
- Updated the operating system and checked basic system information.
- Created the folder and project structure that will be used throughout the SecureWatch Home SOC project.
See [commands.md](commands.md) for the full annotated command reference and [troubleshooting.md](troubleshooting.md) for issues encountered along the way.

## Technologies Used

These are the main tools I used to build this phase of the project:
- **Oracle VirtualBox** – Used to create and run the Ubuntu Server virtual machine.
- **Ubuntu Server LTS** – The operating system running inside the virtual machine (`soc-server`).
- **VirtualBox NAT Networking** – Allowed the server to access the internet while keeping it isolated from my home network.
- **APT** – Ubuntu's package manager, used to install software and keep the system up to date.

## Why I Used a Virtual Machine

I chose to use a virtual machine because it provides a safe environment for learning and experimenting. If I make a mistake or break the system, I can easily restore a snapshot instead of reinstalling the operating system.

Using a VM also keeps my home computer separate from the lab. This is especially important because later phases of this project will include security testing and attack simulations.

## Architecture

See [architecture.md](architecture.md) for the full diagram and network reasoning.

## Security Considerations

While setting up the lab, I followed a few basic security practices:
- **Used NAT networking** to keep the virtual machine isolated from my home network while still allowing internet access.
- **Updated the system first** before installing or configuring any additional software.
- **Chose Ubuntu Server** instead of Ubuntu Desktop because it has fewer installed components, making it lighter and reducing the potential attack surface.

## What I Learned

During this phase, I learned a few important lessons:
- Always check the network connection before installing software. This makes it easier to tell whether a problem is caused by the internet connection or the package manager.
- Recording basic system information at the beginning of the project makes it easier to notice changes later.
- Choosing NAT networking from the start helped keep the lab simple and avoided changing the network configuration later.

## What I Verified

Before continuing to the next phase, I made sure everything was working as expected.
- Oracle VirtualBox was running without issues.
- The Ubuntu Server VM started successfully.
- The hostname was correctly set to `soc-server`.
- The server had internet access.
- The operating system was fully updated.
- The project folders were ready for future phases.

## References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Oracle VirtualBox Networking Modes](https://www.virtualbox.org/manual/ch06.html)

## Next Phase

[Phase 2 — Remote Administration](../phase-02/README.md): enabling and hardening SSH so the server can be managed remotely from the Windows host, the same way a real server would be administered without physical console access.
