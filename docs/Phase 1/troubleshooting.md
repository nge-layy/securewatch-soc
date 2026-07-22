# Phase 1 — Problems Encountered & Solutions

Documenting problems (not just successes) is one of the most valuable habits in SOC and sysadmin work. A record of what broke and how it was fixed is effectively a personal knowledge base — the same function a runbook serves on a real team.

---

### Problem 1: VM had no internet access immediately after install

**Symptom:** `ping 8.8.8.8` failed with 100% packet loss right after the Ubuntu Server installation completed.

**Root Cause:** The network adapter was still initializing / had not yet pulled a DHCP lease from VirtualBox's internal NAT engine.

**Solution:**
```bash
sudo dhclient enp0s3
```
Manually requesting a DHCP lease on the interface resolved the issue immediately. In cases where this doesn't help, restarting the network service is the next step:
```bash
sudo systemctl restart systemd-networkd
```

**Lesson:** Always verify `ip a` shows a valid address *before* assuming a deeper network or firewall problem — many "no internet" issues are just a delayed or failed DHCP handshake.

---

### Problem 2: `apt update` failed with a repository fetch error

**Symptom:**
```
Err:1 http://archive.ubuntu.com/ubuntu jammy InRelease
  Temporary failure resolving 'archive.ubuntu.com'
```

**Root Cause:** DNS resolution was not working inside the guest, even though the raw network connection was fine (confirmed via `ping 8.8.8.8` succeeding while `ping google.com` failed).

**Solution:** Verified `/etc/resolv.conf` was pointing to a valid resolver. Since VirtualBox NAT typically hands out its internal DNS forwarder automatically, restarting the network stack resolved it in this case:
```bash
sudo systemctl restart systemd-resolved
```

**Lesson:** Splitting connectivity testing into "raw IP" (`ping 8.8.8.8`) and "DNS-dependent" (`ping google.com`) checks pinpoints exactly which layer is broken instead of guessing.

---

### Problem 3: Uncertainty about which VirtualBox network mode to use

**Symptom:** Not a technical failure, but a design decision point — NAT, Bridged, and Host-Only all "work," but they behave very differently.

**Resolution:** NAT was selected after weighing the trade-offs documented in [architecture.md](architecture.md). The deciding factor was that the lab needed outbound internet access without exposing the guest to the rest of the home network by default.

**Lesson:** Networking mode is a security decision, not just a connectivity one. Choosing Bridged mode without a specific reason would have unnecessarily expanded the attack surface of the lab from day one.

---

## Summary Table

| Problem | Root Cause | Fix | Prevention |
|---|---|---|---|
| No internet after install | DHCP lease not yet acquired | `dhclient` on interface | Always check `ip a` first |
| `apt update` DNS failure | Broken/slow DNS resolution | Restart `systemd-resolved` | Test raw IP vs DNS separately |
| Network mode uncertainty | Design ambiguity | Chose NAT for isolation + outbound access | Document network decisions upfront |
