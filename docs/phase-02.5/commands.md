# Phase 2.5 - Command Reference

These are the main commands I used to manage users, groups, and permissions on the Ubuntu Server.

---

## 1. Create a New User

```bash
sudo adduser secadmin
```

**Purpose**

Create a new administrator account instead of using the root account for everyday tasks.

---

## 2. Add the User to the sudo Group

```bash
sudo usermod -aG sudo secadmin
```

**Purpose**

Allow the new user to run administrative commands with `sudo`.

**Verify**

```bash
groups secadmin
```

---

## 3. Test sudo Access

```bash
su - secadmin
sudo whoami
```

**Purpose**

Confirm that the new user can use `sudo` successfully.

Expected result:

```text
root
```

---

## 4. View Local User Accounts

```bash
cut -d: -f1 /etc/passwd
```

**Purpose**

Display all local user accounts on the system.

---

## 5. Lock or Remove an Unused Account

```bash
sudo usermod -L username
sudo deluser username
```

**Purpose**

Disable or remove accounts that are no longer needed.

---

## 6. Configure Password Expiration

```bash
sudo chage -M 90 -W 7 secadmin
```

**Purpose**

Set a basic password expiration policy for the administrator account.

**Verify**

```bash
sudo chage -l secadmin
```

---

## 7. Edit the sudo Configuration

```bash
sudo visudo
```

**Purpose**

Safely edit the sudo configuration file.

---

## 8. Check File Ownership

```bash
ls -l /home/secadmin
```

**Purpose**

Verify that files are owned by the correct user.

---

## 9. Change File Ownership

```bash
sudo chown -R secadmin:secadmin /home/secadmin/projects
```

**Purpose**

Assign the correct owner and group to the project files.

---

## 10. Change File Permissions

```bash
chmod 750 /home/secadmin/projects/securewatch-home-soc/scripts
```

**Purpose**

Allow the owner full access while limiting access for other users.

**Verify**

```bash
ls -ld /home/secadmin/projects/securewatch-home-soc/scripts
```
