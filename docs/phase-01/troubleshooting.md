# Phase 1 - Problems & Solutions

During this phase, I ran into a few problems while setting up the lab. Here is how I solved them.

---

## Problem 1: No Internet Connection

**Problem**

The Ubuntu Server VM could not access the internet after installation.

**Solution**

- Checked the network configuration using `ip a`.
- Renewed the DHCP lease.
- Restarted the network service when needed.

```bash
sudo dhclient enp0s3
sudo systemctl restart systemd-networkd
```

**What I Learned**

Always check the network connection first before troubleshooting other issues.

---

## Problem 2: Unable to Update Packages

**Problem**

`apt update` failed because the server could not resolve domain names.

**Solution**

Restarted the DNS resolver service.

```bash
sudo systemctl restart systemd-resolved
```

Then verified both internet connectivity and DNS were working.

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

**What I Learned**

Testing both an IP address and a domain name helps identify whether the problem is the network or DNS.

---

## Problem 3: Choosing the Network Mode

**Problem**

I wasn't sure whether to use NAT, Bridged, or Host-Only networking.

**Solution**

After comparing the options, I chose **NAT** because it provides internet access while keeping the virtual machine isolated from my home network.

**What I Learned**

The network mode is an important security decision. NAT is a good choice for a home lab because it reduces unnecessary exposure.

---

## Summary

| Problem | Solution |
|---------|----------|
| No internet connection | Renewed the DHCP lease and restarted the network service. |
| `apt update` failed | Restarted the DNS resolver and verified connectivity. |
| Choosing a network mode | Selected NAT for internet access and isolation. |
