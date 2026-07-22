# Phase 2 — Problems Encountered & Solutions

---

### Problem 1: `ssh: connect to host 127.0.0.1 port 2222: Connection refused`

**Symptom:** SSH connection attempt from Windows failed immediately with "connection refused."

**Root Cause:** The VirtualBox port-forwarding rule had not yet been saved/applied — the VM was still using the default network configuration with no forwarding rule active.

**Solution:** Re-opened VM Settings → Network → Port Forwarding and confirmed the rule was saved (VirtualBox requires the VM to sometimes be powered off, or the network adapter disabled/re-enabled, for new NAT rules to take effect cleanly). After saving and restarting the VM's network adapter, the connection succeeded.

**Lesson:** "Connection refused" at this stage almost always means the packet never reached a listening service at all — it's a network/forwarding-layer problem, not an SSH authentication problem. Distinguishing "refused" from "timed out" from "authentication failed" narrows troubleshooting significantly.

---

### Problem 2: SSH connection timed out (no "refused" response)

**Symptom:** The Windows SSH client hung for a long period before failing with a timeout, rather than immediately refusing the connection.

**Root Cause:** The guest's firewall (`ufw`, if enabled) was blocking inbound connections on port 22.

**Solution:**
```bash
sudo ufw status
sudo ufw allow 22/tcp
```
Explicitly allowing port 22 through the guest firewall resolved the timeout.

**Lesson:** A timeout (as opposed to an immediate refusal) is a strong signal that a firewall is silently dropping packets rather than actively rejecting the connection — an important distinction when triaging network issues, since the two point to different layers of the stack.

---

### Problem 3: Host key changed warning on a later reconnect

**Symptom:**
```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

**Root Cause:** The VM had been rebuilt/reinstalled between sessions, which generated a new SSH host key — a normal and expected outcome after a fresh OS install, not a sign of compromise in this specific case.

**Solution:** Removed the stale entry from the Windows client's `known_hosts` file and reconnected, accepting the new host key fingerprint after confirming (out of caution) that the rebuild was intentional.

**Lesson:** This warning exists specifically to catch machine-in-the-middle scenarios. In a lab context where you know you just rebuilt the VM, it's safe to accept the new key — but the same warning on a production or unfamiliar system should never be dismissed without verifying the cause first.

---

## Summary Table

| Problem | Root Cause | Fix | Prevention |
|---|---|---|---|
| Connection refused | Port forwarding rule not active | Re-apply/save NAT rule, restart adapter | Verify rule before testing connection |
| Connection timed out | `ufw` blocking port 22 | `ufw allow 22/tcp` | Check firewall rules early in troubleshooting |
| Host key changed warning | VM rebuilt, new host key generated | Remove stale entry from `known_hosts`, verify, reconnect | Always confirm *why* the key changed before accepting |
