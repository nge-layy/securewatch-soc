# SecureWatch Home SOC

---

## What This Project Is

SecureWatch Home SOC is a practice setup that copies what a real Security Operations Center uses. It runs on a secure Linux computer, allows safe remote access, collects all system logs in one place, and prepares those logs so they can be analyzed with a SIEM tool. The whole setup is built with free software: Oracle VirtualBox, Ubuntu Server, built‑in Linux logging tools, and Splunk. Everything runs inside a virtual lab on a Windows 11 computer.

## Lab Environment

I built this home SOC lab on my Windows 11 computer using Oracle VirtualBox. Inside VirtualBox, I created an Ubuntu Server LTS virtual machine named `soc-server`.

To make administration easier, I configured NAT networking with SSH port forwarding so I can manage the server directly from my Windows terminal.

The main log sources used in this phase are:

- `auth.log`
- `syslog`
- `journald`
- `auditd`

These logs will later be forwarded to Splunk for monitoring and analysis.

## Project Roadmap

This project is divided into several phases. Each phase builds on the previous one as I gradually create a functional home Security Operations Center (SOC).

### Phase 1 - Infrastructure Foundation 
- Created the Ubuntu Server virtual machine.
- Configured the network.
- Installed and verified the operating system.

Documentation: [Phase 1](docs/phase-01/README.md)

---

### Phase 2 - Remote Administration 
- Installed and configured SSH.
- Enabled secure remote access from my Windows computer.
- Verified remote administration.

Documentation: [Phase 2](docs/phase-02/README.md)

---

### Phase 3 - Logging & SIEM Preparation 
- Explored Linux log files.
- Configured `auditd`.
- Prepared the server for future Splunk integration.

Documentation: [Phase 3](docs/phase-03/README.md)

---

### Phase 4 - Detection & Alerting
- connected the server to Splunk
- created detection rules
- built dashboards

Documentation: [Phase 4](docs/phase-04/detection.md)

---

### Phase 5 - Attack Simulation
- simulated realistic attacks against the lab to generate security events for analysis.

Documentation: [Phase 5](docs/phase-05/attack-simulation.md)

---

### Phase 6 - Incident Response
- investigated alerts, documented findings, and created incident response reports.

Documentation: [Phase 6](docs/phase-06/investigation.md)

## Repository Structure

```
SecureWatch-Home-SOC/
├── README.md
├── docs/
│   ├── phase-01/
│   │   ├── README.md
│   │   ├── architecture.md
│   │   ├── commands.md
│   │   └── troubleshooting.md
│   ├── phase-02/
│   │   ├── README.md
│   │   ├── architecture.md
│   │   ├── commands.md
│   │   └── troubleshooting.md
│   ├── phase-03/
│   │   ├── README.md
│   │   ├── architecture.md
│   │   ├── logging.md
│   │   ├── commands.md
│   │   └── troubleshooting.md
│   ├── phase-04/
│   │   └── detection.md.md
│   ├── phase-05/
│   │   └── attack-simulation.md.md
│   └── phase-06/
│       └── investigation.md
```

## Skills Demonstrated

`Linux System Administration` · `SSH Hardening` · `Identity & Access Management` · `File Permissions & Ownership` · `sudo / Least Privilege` · `Linux Logging (syslog, journald, auditd)` · `SIEM Log Onboarding` · `Technical Documentation` · `Network Fundamentals (NAT, Port Forwarding)`

## Author

Built and documented as part of an ongoing home-lab SOC portfolio project.

