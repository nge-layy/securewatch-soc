# Phase 1 - Command Reference

These are the main commands I used during Phase 1 to verify the server and prepare it for the next stages of the project.

---

## 1. Check the Network

```bash
ip a
```

**Purpose**

Check that the server has received an IP address and is connected to the network.

---

## 2. Test Internet Connection

```bash
ping -c 4 8.8.8.8
```

**Purpose**

Verify that the server can reach the internet.

---

## 3. Test DNS

```bash
ping -c 4 google.com
```

**Purpose**

Confirm that DNS is working correctly.

---

## 4. Update Package List

```bash
sudo apt update
```

**Purpose**

Download the latest package information from Ubuntu repositories.

---

## 5. Install System Updates

```bash
sudo apt upgrade -y
```

**Purpose**

Update installed packages to the latest versions.

---

## 6. Check System Information

```bash
hostnamectl
uname -r
df -h
free -h
```

**Purpose**

View basic information about the server, including the hostname, kernel version, disk usage, and memory usage.

---

## 7. Create the Project Folder

```bash
mkdir -p ~/projects/securewatch-home-soc/{notes,scripts,configs}
```

**Purpose**

Create folders to organize notes, scripts, and configuration files for future phases.

---

## Verify the Folder Structure

```bash
tree ~/projects/securewatch-home-soc
```

**Expected Output**

```text
securewatch-home-soc
├── configs
├── notes
└── scripts
```
