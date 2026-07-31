# Phase 1 - Architecture & Network Design

## Architecture

```text
Windows 11
    │
    └── Oracle VirtualBox
            │
            └── Ubuntu Server (soc-server)
```

I installed Ubuntu Server inside Oracle VirtualBox on my Windows 11 computer. This virtual machine will be used throughout the SecureWatch Home SOC project.



VirtualBox is a Type 2 hypervisor that runs as an application on top of the host operating system instead of directly on the computer hardware. This model is suitable for a home lab because it does not need dedicated hardware, it allows snapshotting for safe testing, and it keeps the guest operating system completely separate from the host file system.

## Network

```text
Internet
    │
Home Router
    │
Windows 11 Host
    │
VirtualBox (NAT)
    │
Ubuntu Server
```

I selected **NAT** networking because it allows the virtual machine to access the internet while keeping it isolated from other devices on my home network.

In Phase 2, I used port forwarding so I could connect to the server through SSH.

## System Baseline

## System Information

- Hostname: `soc-server`
- Operating System: Ubuntu Server LTS
- Network Mode: NAT
- Shell: `/bin/bash`

I recorded this information after installing Ubuntu so I could compare it with future changes during the project.

## Folder Structure Established in Phase 1

```
soc-server:~$
├── projects/
│   └── securewatch-home-soc/
│       ├── notes/
│       ├── scripts/
│       └── configs/
```

This structure keeps lab-specific scripts and configuration snapshots separate from the rest of the home directory, which becomes important once later phases start generating config backups (SSH, auditd, sudoers) that need to be tracked over time.
