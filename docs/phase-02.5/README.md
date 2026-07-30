# Phase 2.5 - Linux Server Hardening

## Overview

In this phase, I improved the security of my Ubuntu Server by managing users, groups, and permissions. Instead of using the root account for everyday tasks, I created a standard administrator account and used `sudo` when administrative privileges were needed.

I also reviewed user accounts, file permissions, and password settings to make the server more secure.

---

## Objectives

During this phase, I completed the following tasks:

- Created a non-root administrator account.
- Granted administrative access using `sudo`.
- Reviewed users and groups.
- Applied basic password security.
- Checked file and directory permissions.
- Verified that `sudo` was working correctly.

---

## Tools Used

The following tools were used during this phase:

- **Linux Users & Groups** - Manage user accounts and permissions.
- **sudo** - Run administrative commands without logging in as root.
- **chmod** - Change file and directory permissions.
- **chown** - Change file and directory ownership.
- **visudo** - Safely edit the sudo configuration.

---

## Why I Don't Use the Root Account

Instead of logging in as the root user, I use a normal account with `sudo`.

This approach is safer because:

- Administrative commands require confirmation.
- Mistakes are less likely to affect the entire system.
- Administrative actions are recorded in the system logs.

---

## Understanding Linux Permissions

Linux controls access to files and folders using three permission groups:

- **Owner** – The user who owns the file.
- **Group** – Users who belong to the assigned group.
- **Others** – Everyone else.

Each group can have:

- **Read (r)** – View the file.
- **Write (w)** – Modify the file.
- **Execute (x)** – Run the file or enter a directory.

Learning how these permissions work is important for securing a Linux server.

---

## What I Did

During this phase, I:

- Created a dedicated administrator account.
- Added the account to the `sudo` group.
- Reviewed existing user accounts.
- Applied basic password settings.
- Checked file ownership and permissions.
- Verified that `sudo` was working correctly.

---

## Security Considerations

To improve the security of the server, I followed these practices:

- Used the **principle of least privilege**, giving users only the permissions they needed.
- Used individual user accounts instead of sharing the root account.
- Used `visudo` to safely manage sudo permissions.

---

## What I Learned

This phase taught me that:

- Using a normal account with `sudo` is much safer than working as root.
- File permissions are an important part of Linux security.
- User groups make it easier to manage permissions.
- Reviewing users and permissions regularly helps keep the system secure.

---

## Verification

Before moving to the next phase, I confirmed that:

- A non-root administrator account was created.
- The account was added to the `sudo` group.
- `sudo` worked correctly.
- User accounts were reviewed.
- Password settings were applied.
- File permissions and ownership were checked.

---

## References 

- [Linux File Permissions - Red Hat Sysadmin Guide](https://www.redhat.com/sysadmin/linux-file-permissions-explained)
- [sudo and sudoers Manual] (https://www.sudo.ws/docs/man/sudoers.man/)

## Next Phase

In **Phase 3**, I will configure system logging and prepare the server for Splunk integration so security events can be collected and analyzed.
