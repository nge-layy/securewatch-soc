# Phase 1 - Commands

These are the main commands I used during Phase 1 to verify the Ubuntu Server installation and prepare it for the next stages of the SecureWatch Home SOC project.

---

## 1. Check the Network

```bash
ip a
```

**Purpose**

Display the network interfaces and confirm that the server has received an IP address.

---

## 2. Test Internet Connection

```bash
ping -c 4 8.8.8.8
```

**Purpose**

Test internet connectivity using Google's public DNS server.

---

## 3. Test DNS Resolution

```bash
ping -c 4 google.com
```

**Purpose**

Verify that DNS is working by resolving a domain name.

---

## 4. Update the Package List

```bash
sudo apt update
```

**Purpose**

Refresh the package list from the Ubuntu repositories before installing updates.

---

## 5. Install System Updates

```bash
sudo apt upgrade -y
```

**Purpose**

Install the latest available package updates and security patches.

---

## 6. Check System Information

```bash
hostnamectl
```

**Purpose**

Display information about the operating system and hostname.

```bash
uname -r
```

**Purpose**

Display the current Linux kernel version.

```bash
df -h
```

**Purpose**

Check available disk space.

```bash
free -h
```

**Purpose**

View the amount of installed and available memory.

---

## 7. Create the Project Folder

```bash
mkdir -p ~/projects/securewatch-home-soc/{notes,scripts,configs}
```

**Purpose**

Create folders to organize project notes, scripts, and configuration files.

---

## 8. Verify the Folder Structure

If the **tree** package is installed:

```bash
tree ~/projects/securewatch-home-soc
```

Or use:

```bash
ls -R ~/projects/securewatch-home-soc
```

**Purpose**

Verify that the project folders were created successfully.

**Example Output**

```text
securewatch-home-soc
├── configs
├── notes
└── scripts
```

---

## Summary

During this phase I learned how to:

- Check network connectivity.
- Verify internet and DNS access.
- Update an Ubuntu Server.
- View basic system information.
- Organize project files using a simple folder structure.

These basic Linux commands provide the foundation for the next phases of the SecureWatch Home SOC project.
