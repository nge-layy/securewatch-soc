# Phase 1 — Infrastructure Foundation

## Overview

Every SOC — home lab or enterprise — starts with the same question: what is the machine I'm defending, and how well do I actually know it? Phase 1 answers that question. It covers provisioning the virtualized server that will act as the "protected asset" for the rest of the project, verifying it has a stable network identity, and confirming it can reach the internet to pull software and updates.

This phase deliberately avoids shortcuts like pre-built appliance images. Building the VM manually — partitioning decisions, network mode selection, hostname assignment — mirrors how a junior sysadmin or SOC analyst is expected to stand up lab or test infrastructure on the job.

## Objectives

- Install and configure Oracle VirtualBox as the hypervisor on the Windows 11 host
- Provision a new Ubuntu Server virtual machine (`soc-server`)
- Confirm networking (NAT) and internet connectivity from inside the guest
- Perform initial system updates and verify system information
- Establish the baseline folder/project structure used throughout the lab

## Technologies Used

| Tool | Purpose |
|---|---|
| Oracle VirtualBox | Type-2 hypervisor hosting the guest VM |
| Ubuntu Server (LTS) | Guest operating system — the "soc-server" |
| VirtualBox NAT networking | Provides outbound internet access to the guest without exposing it to the host LAN |
| APT | Ubuntu's package manager, used for system updates |

## Why a Virtual Machine Instead of Bare Metal?

A VM gives an isolated, disposable environment. If a later phase (hardening, log tampering tests, or eventual attack simulation) breaks the system, the fix is reverting a snapshot rather than reinstalling an OS on physical hardware. Isolation also matters for safety: a home lab used for attack simulation should never share a network segment directly with the host's other traffic without a deliberate decision to do so.

## Architecture

See [architecture.md](architecture.md) for the full diagram and network reasoning.

## Step-by-Step Summary

1. Installed Oracle VirtualBox on the Windows 11 host.
2. Downloaded the Ubuntu Server ISO and created a new VM with dedicated CPU, RAM, and disk allocations.
3. Selected **NAT** as the network adapter mode so the VM could reach the internet through the host's connection without being directly reachable from the host's LAN.
4. Completed the Ubuntu Server installation, setting the hostname to `soc-server`.
5. Logged in and verified the network interface received an address via DHCP from VirtualBox's internal NAT engine.
6. Ran `apt update && apt upgrade` to bring the base OS to a current patch level before any other configuration.
7. Captured baseline system information (kernel version, disk layout, memory) for future reference.
8. Created the project folder structure to keep configuration notes and scripts organized as the lab grows.

See [commands.md](commands.md) for the full annotated command reference and [troubleshooting.md](troubleshooting.md) for issues encountered along the way.

## Security Considerations

- **NAT vs. Bridged networking:** NAT was chosen deliberately over Bridged mode. Bridged mode would place `soc-server` directly on the home LAN with its own IP, making it reachable by every other device on the network. NAT keeps the VM's exposure limited to the host itself, which matters once the lab starts hosting intentionally vulnerable services in later phases.
- **Patching before configuring:** Updating packages before installing or configuring any service reduces the chance of building on top of a package with a known vulnerability.
- **Minimal install footprint:** Ubuntu Server (no GUI) was used instead of Desktop to reduce the attack surface — fewer installed packages means fewer potential vulnerabilities and a smaller footprint to monitor.

## Lessons Learned

- Confirming network connectivity *before* attempting package installation saves significant troubleshooting time — a "no internet" problem looks identical to a "broken package mirror" problem until you isolate it.
- Recording baseline system information (kernel version, disk usage) at the start of a project makes it much easier to spot drift or unexpected changes later, which is exactly the mindset a SOC analyst needs when establishing a system baseline for anomaly detection.
- Deciding on the network mode early (NAT) avoided having to re-architect networking after services were already configured.

## Verification Checklist

- [x] VirtualBox installed and functional on host
- [x] `soc-server` VM created and boots to login prompt
- [x] Hostname correctly set to `soc-server`
- [x] Network interface has a valid IP via NAT/DHCP
- [x] Outbound internet connectivity confirmed (ping + package fetch)
- [x] System fully updated (`apt update && apt upgrade`)
- [x] Project folder structure created

## References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Oracle VirtualBox Networking Modes](https://www.virtualbox.org/manual/ch06.html)

## Next Phase

➡️ [Phase 2 — Remote Administration](../phase-02/README.md): enabling and hardening SSH so the server can be managed remotely from the Windows host, the same way a real server would be administered without physical console access.
