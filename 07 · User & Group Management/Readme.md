# 07 · User & Group Management

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Create, modify, lock, audit, and remove users and groups — including service accounts, password policies, and access reviews required in regulated environments.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [The Four Key Files](#2-the-four-key-files)
3. [Creating Users — `useradd` / `adduser`](#3-creating-users--useradd--adduser)
4. [Passwords — `passwd` & `chpasswd`](#4-passwords--passwd--chpasswd)
5. [Password Aging — `chage`](#5-password-aging--chage)
6. [Modifying Users — `usermod`](#6-modifying-users--usermod)
7. [Deleting Users — `userdel`](#7-deleting-users--userdel)
8. [Groups — `groupadd`, `groupmod`, `groupdel`, `gpasswd`](#8-groups)
9. [Querying & Auditing Users](#9-querying--auditing-users)
10. [Switching Users — `su`](#10-switching-users--su)
11. [Service Accounts](#11-service-accounts)
12. [Defaults: `/etc/login.defs`, `/etc/default/useradd`, `/etc/skel`](#12-defaults)
13. [Resource Limits — `ulimit` & `limits.conf`](#13-resource-limits--ulimit--limitsconf)
14. [Password Policy (PAM) & Account Lockout](#14-password-policy-pam--account-lockout)
15. [Centralized Identity (LDAP / AD / SSSD)](#15-centralized-identity-ldap--ad--sssd)
16. [Real-World Scenarios](#16-real-world-scenarios)
17. [Interview Questions](#17-interview-questions)
18. [Cheat Sheet](#18-cheat-sheet)

---

## 1. Core Concepts

| Term | Meaning |
|------|---------|
| **UID** | User ID number (kernel uses numbers, not names) |
| **GID** | Group ID number |
| **Primary group** | Default group on files the user creates (1 per user) |
| **Secondary / supplementary groups** | Extra groups for access (docker, wheel) |
| **root** | UID 0 — full control |
| **System users** | UID 1–999 — run services (nginx, mysql) |
| **Regular users** | UID 1000+ — humans |
| **nologin shell** | `/sbin/nologin` — account can't get an interactive shell |

---

## 2. The Four Key Files

### `/etc/passwd` — account info (world-readable)

```
manoj:x:1001:1001:Manoj Kumar,SRE:/home/manoj:/bin/bash
  │   │  │    │        │               │           │
 user pw UID GID    comment (GECOS)   home        shell
      (x = password in /etc/shadow)
```

### `/etc/shadow` — password hashes (root only)

```
manoj:$6$salt$hash...:20000:1:90:7:14:20400:
  │        │           │    │  │  │  │    │
 user   hash      lastchg min max warn inactive expire
```

| Hash prefix | Algorithm |
|-------------|-----------|
| `$1$` | MD5 (weak) |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 (common) |
| `$y$` | yescrypt (modern Debian/RHEL 9) |
| `!` or `!!` | Locked / no password set |
| `*` | Login disabled (system accounts) |

### `/etc/group`

```
docker:x:994:manoj,jenkins
  │    │  │      │
 name  pw GID  members (secondary)
```

### `/etc/gshadow` — group passwords & admins (root only)

> **Never edit directly with a normal editor.** Use commands, or `vipw` / `vigr` (lock + safe edit).

```bash
sudo vipw          # Edit /etc/passwd safely
sudo vipw -s       # Edit /etc/shadow safely
sudo vigr          # Edit /etc/group safely
sudo pwck          # Check passwd/shadow consistency
sudo grpck         # Check group/gshadow consistency
```

---

## 3. Creating Users — `useradd` / `adduser`

```bash
sudo useradd -m -s /bin/bash manoj                       # Basic human user
sudo useradd -m -s /bin/bash -c "Manoj Kumar - SRE" -G wheel,docker manoj
sudo useradd -m -u 1500 -g devops -G docker,wheel -e 2026-12-31 contractor1
sudo useradd -r -s /sbin/nologin -d /opt/app -M appsvc    # Service account
sudo useradd -D                                           # Show defaults
sudo adduser manoj                                        # Debian interactive wrapper
```

| Option | What it does |
|--------|--------------|
| `-m` | Create home directory from skel |
| `-M` | Do not create home directory |
| `-d DIR` | Set custom home directory path |
| `-s SHELL` | Set user's login shell |
| `-c "text"` | Comment / full name (GECOS) |
| `-u UID` | Set specific user ID number |
| `-g GROUP` | Set primary group (must exist) |
| `-G g1,g2` | Add supplementary (secondary) groups |
| `-r` | Create system account, low UID |
| `-e YYYY-MM-DD` | Account expires on this date |
| `-f N` | Disable N days after password expiry |
| `-k DIR` | Use alternate skeleton directory |
| `-N` | Don't create same-name group |
| `-U` | Create group with same name |
| `-o` | Allow duplicate (non-unique) UID |
| `-p HASH` | Set pre-encrypted password hash |
| `-D` | Show or change useradd defaults |

> **Tip:** On Ubuntu `useradd` without `-m -s` creates no home and `/bin/sh`. Always pass `-m -s /bin/bash`, or use `adduser`.

---

## 4. Passwords — `passwd` & `chpasswd`

```bash
passwd                          # Change your own password
sudo passwd manoj               # Set another user's password
sudo passwd -l manoj            # Lock account
sudo passwd -u manoj            # Unlock account
sudo passwd -e manoj            # Force change at next login
sudo passwd -d manoj            # Delete password (⚠️ passwordless)
sudo passwd -S manoj            # Status: P=set, L=locked, NP=none
echo "manoj:TempP@ss123" | sudo chpasswd        # Non-interactive (automation)
sudo chpasswd < users_passwords.txt              # Bulk
openssl passwd -6 'MyPass'                       # Generate SHA-512 hash
```

| Option | What it does |
|--------|--------------|
| `-l` | Lock password, prefix hash with `!` |
| `-u` | Unlock a previously locked password |
| `-e` | Expire password, force change next |
| `-d` | Delete password, make it empty |
| `-S` | Show password status for account |
| `-n / -x / -w` | Min days / max days / warn days |
| `--stdin` | Read password from stdin (RHEL) |

> `passwd -l` locks only **password** logins — SSH keys still work! To fully block: `usermod -L -e 1 user` + change shell to nologin (see Offboarding in topic 10).

---

## 5. Password Aging — `chage`

```bash
sudo chage -l manoj                      # Show aging info
sudo chage -M 90 -m 1 -W 7 manoj         # Max 90d, min 1d, warn 7d
sudo chage -d 0 manoj                    # Force change at next login
sudo chage -E 2026-12-31 contractor1     # Account expiry date
sudo chage -E -1 manoj                   # Remove expiry
sudo chage -I 14 manoj                   # Lock 14d after pw expiry
```

| Option | What it does |
|--------|--------------|
| `-l` | List current password aging info |
| `-M N` | Max days before password must change |
| `-m N` | Min days between password changes |
| `-W N` | Warn N days before expiry |
| `-I N` | Inactive days after expiry, then lock |
| `-E DATE` | Account expiration date (-1 none) |
| `-d 0` | Set last change 0, force reset |

---

## 6. Modifying Users — `usermod`

```bash
sudo usermod -aG docker manoj            # ADD to group (keep others) ← most used
sudo usermod -G docker manoj             # ⚠️ REPLACES all secondary groups
sudo usermod -s /bin/zsh manoj           # Change shell
sudo usermod -s /sbin/nologin olduser    # Disable shell access
sudo usermod -L manoj                    # Lock
sudo usermod -U manoj                    # Unlock
sudo usermod -e 2026-12-31 manoj         # Expiry date
sudo usermod -e 1 manoj                  # Expire immediately (disables all auth incl. SSH keys)
sudo usermod -l newname oldname          # Rename login
sudo usermod -d /data/home/manoj -m manoj   # Move home dir
sudo usermod -u 2001 manoj               # Change UID (fix file ownership after!)
sudo usermod -g devops manoj             # Change primary group
sudo usermod -c "Manoj K - Lead SRE" manoj
sudo gpasswd -d manoj docker             # Remove from one group
```

| Option | What it does |
|--------|--------------|
| `-aG` | Append to groups, keep existing ones |
| `-G` | Set groups exactly, replaces existing |
| `-g` | Change primary group of user |
| `-s` | Change user's login shell |
| `-L` / `-U` | Lock / unlock user's password |
| `-e DATE` | Set account expiry date |
| `-l NEW` | Rename the login name |
| `-d DIR -m` | Set new home, move contents |
| `-u UID` | Change the user ID number |
| `-c` | Change comment / full name |

> After being added to a group, the user must **log out and back in** (or `newgrp docker`) for it to take effect.

---

## 7. Deleting Users — `userdel`

```bash
sudo userdel olduser                # Remove account, keep home
sudo userdel -r olduser             # Remove account + home + mail spool
sudo userdel -f olduser             # Force, even if logged in
sudo pkill -KILL -u olduser         # Kill their processes first
sudo find / -xdev -nouser 2>/dev/null   # Find orphaned files after deletion
```

| Option | What it does |
|--------|--------------|
| `-r` | Also delete home and mail spool |
| `-f` | Force removal even if logged in |

> **In regulated/banking environments:** lock + expire first, archive home, and delete only after the retention period — auditors want evidence.

---

## 8. Groups

```bash
sudo groupadd devops                       # Create group
sudo groupadd -g 3000 sre                  # With specific GID
sudo groupadd -r appgrp                    # System group
sudo groupmod -n platform devops           # Rename group
sudo groupmod -g 3100 sre                  # Change GID
sudo groupdel oldgroup                     # Delete group
sudo gpasswd -a manoj sre                  # Add user to group
sudo gpasswd -d manoj sre                  # Remove user from group
sudo gpasswd -M alice,bob,manoj sre        # Set full member list
sudo gpasswd -A manoj sre                  # Make manoj group admin
newgrp docker                              # Activate new group in current shell
getent group sre                           # Show group + members
```

| Option | What it does |
|--------|--------------|
| `groupadd -g` | Assign a specific group ID |
| `groupadd -r` | Create system group, low GID |
| `groupmod -n` | Rename the group to new name |
| `gpasswd -a` | Add a single user to group |
| `gpasswd -d` | Remove a single user from group |
| `gpasswd -M` | Replace entire member list at once |
| `gpasswd -A` | Set group administrators list |

### Important Groups

| Group | Grants |
|-------|--------|
| `wheel` (RHEL) / `sudo` (Ubuntu) | sudo access |
| `docker` | Docker socket (≈ root!) |
| `adm` / `systemd-journal` | Read system logs |
| `ssh-users` (custom) | Allowed in `sshd_config AllowGroups` |

---

## 9. Querying & Auditing Users

```bash
id manoj                       # UID, GID, groups
groups manoj                   # Group names
getent passwd manoj            # Works for local + LDAP/AD users ← preferred
getent group docker
finger manoj / pinky manoj     # User info (if installed)
who / w                        # Logged-in users
last manoj                     # Login history
lastlog                        # Last login of every user
lastlog -b 90                  # Not logged in for 90+ days ← stale accounts
sudo lastb | head              # Failed logins
sudo faillock --user manoj     # Failed attempt counter (RHEL 8+)
```

**Audit one-liners**
```bash
awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd     # Human users
awk -F: '$3 == 0 {print $1}' /etc/passwd                      # UID 0 accounts (should be only root!)
awk -F: '$2 == "" {print $1}' /etc/shadow                     # Empty passwords 🚨
awk -F: '$7 !~ /(nologin|false)/ {print $1, $7}' /etc/passwd  # Users with login shells
getent group wheel sudo                                       # Who has sudo via group
cut -d: -f1 /etc/passwd | sort | uniq -d                      # Duplicate usernames
```

---

## 10. Switching Users — `su`

```bash
su - manoj                    # Switch with full login env ← correct way
su manoj                      # Switch, keep current env (not recommended)
su -                          # Become root (needs root password)
sudo -i                       # Root login shell via sudo (preferred)
sudo -u appsvc -i             # Shell as service account
sudo -u appsvc whoami         # One command as another user
su -s /bin/bash -c "cmd" appsvc   # Run as nologin service user
exit                          # Return to previous user
```

| Option | What it does |
|--------|--------------|
| `-` / `-l` | Full login shell, load user env |
| `-c "cmd"` | Run single command, then exit |
| `-s SHELL` | Use this shell instead |

---

## 11. Service Accounts

Best practice for apps (nginx, jenkins, custom apps):

```bash
sudo useradd --system --no-create-home --shell /sbin/nologin --home-dir /opt/myapp myapp
sudo chown -R myapp:myapp /opt/myapp /var/log/myapp
```

- No password, no interactive shell, no sudo.
- Run the service with `User=myapp` in its systemd unit.
- One service = one account (least privilege, clear audit trail).

---

## 12. Defaults

| File | Controls |
|------|----------|
| `/etc/login.defs` | UID/GID ranges, `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`, `UMASK`, `ENCRYPT_METHOD`, `CREATE_HOME` |
| `/etc/default/useradd` | Default shell, home base, skel, expiry |
| `/etc/skel/` | Files copied into every new home (`.bashrc`, `.bash_profile`) |

```bash
grep -E '^(PASS_|UID_|UMASK|ENCRYPT)' /etc/login.defs
sudo useradd -D -s /bin/bash          # Change default shell for new users
```

---

## 13. Resource Limits — `ulimit` & `limits.conf`

```bash
ulimit -a                 # Show all limits for current shell
ulimit -n                 # Max open files (soft)
ulimit -Hn                # Hard limit for open files
ulimit -n 65535           # Raise for current session
ulimit -u                 # Max user processes
cat /proc/$(pidof nginx | cut -d' ' -f1)/limits   # Limits of a running process
```

| Option | What it does |
|--------|--------------|
| `-a` | Show all current limits |
| `-n` | Max number of open files |
| `-u` | Max processes for this user |
| `-c` | Max core dump file size |
| `-H` / `-S` | Hard limit / soft limit |

**Persistent: `/etc/security/limits.conf` or `/etc/security/limits.d/90-app.conf`**
```
appsvc   soft   nofile   65535
appsvc   hard   nofile   65535
@devops  hard   nproc    4096
```

> systemd services ignore `limits.conf` — use `LimitNOFILE=65535` in the unit file.

---

## 14. Password Policy (PAM) & Account Lockout

**Password complexity — `/etc/security/pwquality.conf`**
```
minlen = 14
dcredit = -1    # at least 1 digit
ucredit = -1    # 1 uppercase
lcredit = -1    # 1 lowercase
ocredit = -1    # 1 special char
retry = 3
```

**Lockout after failed logins — `/etc/security/faillock.conf` (RHEL 8+/Ubuntu 22+)**
```
deny = 5
unlock_time = 900
```

```bash
sudo faillock --user manoj             # Show failures
sudo faillock --user manoj --reset     # Unlock account after lockout
sudo pam_tally2 -u manoj -r            # Older systems
```

---

## 15. Centralized Identity (LDAP / AD / SSSD)

In enterprises, users usually come from **Active Directory / LDAP** via **SSSD**, not `/etc/passwd`.

```bash
getent passwd jdoe@corp.example.com       # AD user lookup
id jdoe@corp.example.com
realm list                               # Joined domain info
sudo realm join corp.example.com -U admin
sudo sss_cache -E                        # Clear SSSD cache (group change not visible)
systemctl status sssd
```

> That's why `getent` beats `grep /etc/passwd` — it queries **all** sources in `/etc/nsswitch.conf`.

---

## 16. Real-World Scenarios

**Onboard a new engineer**
```bash
sudo useradd -m -s /bin/bash -c "Priya Shah - SRE" -G wheel,docker priya
sudo passwd -e priya                                  # Or set up SSH key only
sudo mkdir -m 700 /home/priya/.ssh
echo "ssh-ed25519 AAAA... priya@laptop" | sudo tee /home/priya/.ssh/authorized_keys
sudo chmod 600 /home/priya/.ssh/authorized_keys
sudo chown -R priya:priya /home/priya/.ssh
```

**Temporary contractor access (auto-expires)**
```bash
sudo useradd -m -s /bin/bash -e 2026-12-31 -c "Vendor - Ticket CHG12345" vendor1
```

**Shared team directory**
```bash
sudo groupadd devops
sudo usermod -aG devops alice && sudo usermod -aG devops bob
sudo mkdir /srv/devops && sudo chgrp devops /srv/devops && sudo chmod 2770 /srv/devops
```

**"User added to docker group but still permission denied"**
```bash
id manoj            # Check group listed
# User must re-login, or run: newgrp docker
```

**Quarterly access review**
```bash
lastlog -b 90 | awk 'NR>1 && $0 !~ /Never/ {print $1}'
getent group wheel
```

---

## 17. Interview Questions

1. **`/etc/passwd` vs `/etc/shadow`?** passwd = account info (readable by all); shadow = hashes & aging (root only).
2. **`usermod -G` vs `usermod -aG`?** `-G` replaces groups; `-aG` appends.
3. **Primary vs secondary group?** Primary owns new files; secondary grants extra access.
4. **UID 0 accounts other than root?** Security red flag — audit with `awk -F: '$3==0' /etc/passwd`.
5. **How to force password change at next login?** `chage -d 0 user` or `passwd -e user`.
6. **User locked but still logs in — why?** SSH key auth bypasses password lock; expire account (`usermod -e 1`) and remove keys.
7. **Why `getent` instead of `cat /etc/passwd`?** Includes LDAP/AD users via NSS.
8. **Why is `docker` group dangerous?** Docker socket access = root-equivalent.

---

## 18. Cheat Sheet

```bash
useradd -m -s /bin/bash -c -G -u -e | useradd -r -M -s /sbin/nologin
passwd -l -u -e -S | chpasswd | chage -l -M -E -d 0
usermod -aG -s -L -U -e -l -d -m
userdel -r | pkill -u | find / -nouser
groupadd -g | groupmod -n | groupdel | gpasswd -a -d -M | newgrp
id | groups | getent passwd/group | last | lastlog -b 90 | faillock --reset
su - | sudo -i | sudo -u user -i
vipw | vigr | pwck | grpck
ulimit -n | /etc/security/limits.conf | LimitNOFILE=
```
