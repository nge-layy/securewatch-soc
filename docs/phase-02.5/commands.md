# Phase 2.5 — Command Reference

---

### 1. Create a new administrative user

```bash
sudo adduser secadmin
```

**Why:** `adduser` (the higher-level, interactive Debian/Ubuntu tool) creates the user's home directory, sets ownership correctly, and prompts for a password and basic account info in one guided flow — reducing the chance of a half-configured account compared to the lower-level `useradd`.

**Expected output (excerpt):**
```
Adding user `secadmin' ...
Adding new group `secadmin' (1001) ...
Adding new user `secadmin' (1001) with group `secadmin' ...
Creating home directory `/home/secadmin' ...
```

---

### 2. Add the new user to the `sudo` group

```bash
sudo usermod -aG sudo secadmin
```

**Why:** Membership in the `sudo` group is what grants a user the ability to run commands as root via `sudo`, based on the default rule in `/etc/sudoers`. The `-a` (append) flag is critical — without it, `usermod -G` would *replace* all of the user's existing group memberships with just the one specified, silently removing them from any other groups they belonged to.

**Verification:**
```bash
groups secadmin
```
```
secadmin : secadmin sudo
```

---

### 3. Verify sudo access works

```bash
su - secadmin
sudo whoami
```

**Why:** Confirms the privilege escalation path actually functions end-to-end, not just that the group membership exists on paper.

**Expected output:**
```
root
```

If `sudo whoami` returns `root`, the user can successfully escalate privileges when authorized.

---

### 4. Review all local user accounts

```bash
cat /etc/passwd | cut -d: -f1
```

**Why:** Lists every account on the system by username. Reviewing this list is how unnecessary or unexpected accounts get identified — every entry should have a clear, known reason for existing.

---

### 5. Lock or remove an unnecessary account

```bash
sudo usermod -L unused_account      # lock (disable password login)
sudo deluser unused_account         # remove entirely, if appropriate
```

**Why:** Locking (`-L`) disables password-based login without deleting the account or its files — useful when you're not yet sure it's safe to fully remove. `deluser` is used only once it's confirmed the account and its data are no longer needed, since it's a destructive, harder-to-reverse action.

---

### 6. Set a basic password aging / complexity policy

```bash
sudo chage -M 90 -W 7 secadmin
```

**Why:** `-M 90` sets the maximum password age to 90 days, forcing periodic password rotation. `-W 7` warns the user 7 days before expiration. This is a baseline password policy control — in an enterprise environment this would typically be centrally enforced (e.g., via PAM policy or an identity provider), but applying it manually here demonstrates the same underlying concept.

**Verification:**
```bash
sudo chage -l secadmin
```

---

### 7. Edit sudoers safely

```bash
sudo visudo
```

**Why:** `visudo` opens `/etc/sudoers` in a syntax-checked editing session — it will refuse to save a file with invalid syntax, which prevents the single most dangerous mistake possible in this file: saving a broken sudoers configuration that locks out all administrative (`sudo`) access system-wide, with no easy way back in short of console-level recovery.

---

### 8. Audit file ownership

```bash
ls -l /home/secadmin
```

**Why:** Confirms that files are owned by the correct user/group. A file owned by the wrong user (e.g., left over from a previous account, or accidentally created as `root` via a stray `sudo` command) can create both access problems and privilege-escalation risks.

**Expected output (excerpt):**
```
-rw-r--r-- 1 secadmin secadmin  220 Jan  1 00:00 .bash_logout
```

---

### 9. Correct ownership

```bash
sudo chown secadmin:secadmin /home/secadmin/projects -R
```

**Why:** `chown` changes both the owning user and group in one command. The `-R` flag applies the change recursively to every file and subdirectory — necessary when an entire directory tree was created or modified under the wrong ownership (a common outcome of running setup commands with `sudo` without thinking through resulting file ownership).

---

### 10. Correct permissions

```bash
chmod 750 /home/secadmin/projects/securewatch-home-soc/scripts
```

**Why:** `750` grants the owner full access (`rwx`), the group read+execute (`r-x`), and denies all access to everyone else. This is an appropriate baseline for a scripts directory: the owner can edit and run scripts, trusted group members can execute them, and no other local user can read or run them at all.

**Verification:**
```bash
ls -ld /home/secadmin/projects/securewatch-home-soc/scripts
```
```
drwxr-x--- 2 secadmin secadmin 4096 Jan  1 00:00 scripts
```
