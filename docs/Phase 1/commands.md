# Phase 1 — Command Reference

Every command below includes *why* it was run, not just its syntax. Understanding intent is what separates following a checklist from actually administering a system.

---

### 1. Check network interface and confirm the VM received an address

```bash
ip a
```

**Why:** Before anything else, confirm the guest actually has network connectivity. Without a valid IP, every subsequent step (updates, package installs, SSH) will fail in ways that look unrelated to the real cause.

**Expected output (excerpt):**
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
```

A `10.0.2.x` address confirms the VM is using VirtualBox's default NAT network range and has successfully obtained a DHCP lease from the virtual NAT engine.

---

### 2. Test outbound connectivity

```bash
ping -c 4 8.8.8.8
```

**Why:** Confirms Layer 3 (IP) connectivity to the internet independent of DNS. If this fails but the interface has an IP, the problem is routing/NAT — not DNS.

**Expected output:**
```
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

---

### 3. Test DNS resolution

```bash
ping -c 4 google.com
```

**Why:** Confirms name resolution is working, not just raw IP connectivity. A system can have working internet access but broken DNS (or vice versa) — testing both isolates which layer to troubleshoot if something fails.

---

### 4. Update the package index

```bash
sudo apt update
```

**Why:** This refreshes the local index of available packages and their versions from Ubuntu's repositories. It does **not** install anything — it just makes sure the system knows what the latest available versions are. Skipping this step risks installing outdated or vulnerable package versions.

---

### 5. Upgrade installed packages

```bash
sudo apt upgrade -y
```

**Why:** Applies the latest available versions of already-installed packages, including security patches. Running this immediately after install ensures the lab isn't built on a foundation with known, patched vulnerabilities still present. The `-y` flag auto-confirms prompts — acceptable here because this is a fresh, low-risk lab environment; on production systems, reviewing the upgrade list first is safer practice.

---

### 6. Capture system information

```bash
hostnamectl
uname -r
df -h
free -h
```

**Why:** `hostnamectl` confirms the hostname and OS identity are set correctly. `uname -r` records the exact kernel version — useful later when checking a kernel version against known CVEs. `df -h` and `free -h` establish disk and memory baselines so any unusual future consumption (e.g., a log flood or a compromise dropping files) stands out against a known-normal state.

**Expected output (`hostnamectl`, excerpt):**
```
 Static hostname: soc-server
       Icon name: computer-vm
         Chassis: vm
      Machine ID: ...
         Boot ID: ...
  Virtualization: oracle
Operating System: Ubuntu 22.04 LTS
          Kernel: Linux 5.15.0-generic
    Architecture: x86-64
```

---

### 7. Create the project folder structure

```bash
mkdir -p ~/projects/securewatch-home-soc/{notes,scripts,configs}
```

**Why:** `mkdir -p` creates the full nested path in one command and does not error if a parent directory already exists, which makes it safe to re-run. Organizing notes, scripts, and config backups into dedicated subfolders from day one avoids a cluttered home directory once the project grows across multiple phases.

**Verification:**
```bash
tree ~/projects/securewatch-home-soc
```
```
projects/securewatch-home-soc
├── configs
├── notes
└── scripts
```
