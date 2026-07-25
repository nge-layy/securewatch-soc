# Phase 2.5 — Linux Server Hardening

## Overview

Getting remote access working (Phase 2) answers "can someone get in?" Hardening answers the more important question: "once they're in, what can they actually do?" Phase 2.5 focuses on identity and access management on `soc-server` — creating a properly scoped administrative user, understanding and controlling group membership, applying the principle of least privilege through `sudo`, and correcting the permission and ownership mistakes that are among the most common root causes of real-world breaches.

This phase treats every user, group, and permission bit as a deliberate security decision rather than a default to be accepted.

## Objectives

- Create a non-root administrative user for day-to-day work
- Understand and correctly apply Linux's user/group/permission model
- Grant administrative privileges through `sudo` rather than direct root login
- Remove unnecessary default access and review existing accounts
- Establish a basic password policy
- Audit and correct file/directory ownership and permissions

## Why Not Just Use Root?

Logging in and working as `root` directly is one of the most common and most dangerous shortcuts in Linux administration. Every command run as root executes with unrestricted power over the entire system — a single mistyped `rm -rf` or a compromised SSH session has no safety net. Using a standard user with `sudo` for specific administrative commands means:

- Actions requiring elevated privileges are deliberate and logged (`sudo` logs every invocation to `auth.log`, which becomes directly relevant in Phase 3).
- Mistakes made as a standard user are contained to that user's own permissions rather than the whole system.
- If SSH credentials are ever compromised, the attacker doesn't automatically inherit root — they still have to escalate, which is a second barrier, not zero barriers.

## How Linux Permissions Work

Every file and directory on a Linux system carries three sets of permissions — for its **owner**, its **group**, and **everyone else** — each of which can independently allow **read**, **write**, and **execute**.

```
-rwxr-x---  1 alice  socteam   1024  Jan  1 00:00  deploy.sh
 │││ │││ │││
 │││ │││ └──┴─ others: no permissions
 │││ └───────  group (socteam): read + execute
 └───────────  owner (alice): read + write + execute
```

| Symbol | Meaning (files) | Meaning (directories) |
|---|---|---|
| `r` | Read file contents | List directory contents |
| `w` | Modify file contents | Create/delete files within |
| `x` | Execute the file | Enter (`cd` into) the directory |

Getting this wrong is one of the most common misconfigurations in real environments — for example, log files that are world-readable can leak sensitive information, and world-writable scripts can be tampered with by any local user.

## How Groups Work

Groups let permissions be assigned to a set of users at once instead of managing every file individually per user. In this project, group membership becomes especially important in Phase 3, where the `splunk` service account is added to the `adm` group specifically to grant it read access to system logs — without granting it any other privileges on the system.

## How `sudo` Works

`sudo` ("superuser do") allows a permitted user to run a specific command with elevated privileges, governed by rules in `/etc/sudoers` (edited safely via `visudo`, which validates syntax before saving to prevent a broken sudoers file from locking out all administrative access). This is the mechanism that lets an admin account behave like root *only* for the duration and scope of a specific command, rather than for an entire logged-in session.

## Step-by-Step Summary

1. Created a dedicated administrative user (separate from any default install account).
2. Added the new user to the `sudo` group to grant administrative capability without direct root login.
3. Reviewed existing local accounts and removed/disabled any unnecessary default access.
4. Set and verified a password policy (minimum complexity/aging requirements).
5. Audited ownership (`chown`) and permissions (`chmod`) across key project and configuration directories.
6. Verified `sudo` access worked correctly and was properly logged.

Full command-by-command detail is in [commands.md](commands.md), and the deeper permissions/ownership walkthrough is in [hardening.md](hardening.md).

## Common Permission Mistakes (and Why They Matter)

| Mistake | Why It's a Problem |
|---|---|
| `chmod 777` "to make it work" | Grants read/write/execute to *everyone* on the system — any local user or compromised process can modify the file |
| Running services as `root` unnecessarily | A vulnerability in the service now grants root-level access to an attacker, instead of a limited service-account's permissions |
| Leaving default/unused accounts enabled | Every extra account is an extra potential authentication target |
| Editing `/etc/sudoers` directly with a text editor | A syntax error can lock out all `sudo` access system-wide; `visudo` prevents saving invalid syntax |
| Sharing one account among multiple "admins" | Destroys accountability — logs can't show *which* person ran a given command |

## Security Considerations

- **Least privilege** is the guiding principle throughout this phase: every user and every permission grant is scoped to exactly what's needed, nothing more.
- **Accountability**: individual accounts (rather than shared logins) ensure `auth.log` entries can be tied to a specific identity — essential for any future incident investigation.
- **Fail-safe editing**: using `visudo` instead of directly editing `/etc/sudoers` avoids a class of self-inflicted lockouts that are a genuine (and common) risk in real environments.

## Lessons Learned

- Least privilege isn't just a security buzzword — it's directly testable: for every account and permission on the box, the question "does this actually need to be true?" almost always reveals something that can be tightened.
- `visudo`'s syntax validation is not optional friction — it exists because a broken sudoers file with no other root access is a genuinely painful (and avoidable) way to get locked out of your own server.
- Ownership and permissions issues compound quietly. A directory made world-writable "just to test something" is easy to forget about and easy to miss during a later review unless permissions audits become a habit, not a one-time event.

## Verification Checklist

- [x] Dedicated non-root administrative user created
- [x] Administrative user added to `sudo` group
- [x] `sudo` access tested and confirmed logged in `auth.log`
- [x] Unnecessary default accounts reviewed/removed
- [x] Basic password policy applied
- [x] File/directory ownership and permissions audited and corrected

## References

- [Linux File Permissions — Red Hat Sysadmin Guide](https://www.redhat.com/sysadmin/linux-file-permissions-explained)
- [sudo and sudoers Manual](https://www.sudo.ws/docs/man/sudoers.man/)

## Next Phase

➡️ [Phase 3 — Logging & SIEM Preparation](../phase-03/README.md): with identity, access, and permissions under control, the next step is making sure everything that happens on this server is actually recorded — and that a future SIEM can read those records safely.
