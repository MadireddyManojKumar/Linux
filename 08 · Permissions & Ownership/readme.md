# 08 · Permissions & Ownership

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Control who can read, write and execute what — and debug every "Permission denied" in minutes (standard perms, special bits, umask, ACLs, attributes, SELinux).

---

## Table of Contents

1. [Reading Permissions](#1-reading-permissions)
2. [Permission Meaning: Files vs Directories](#2-permission-meaning-files-vs-directories)
3. [Numeric (Octal) Notation](#3-numeric-octal-notation)
4. [chmod — Change Permissions](#4-chmod--change-permissions)
5. [chown & chgrp — Change Ownership](#5-chown--chgrp--change-ownership)
6. [Special Permissions: SUID, SGID, Sticky Bit](#6-special-permissions-suid-sgid-sticky-bit)
7. [umask — Default Permissions](#7-umask--default-permissions)
8. [ACLs — Fine-Grained Access](#8-acls--fine-grained-access)
9. [File Attributes — `chattr` / `lsattr`](#9-file-attributes--chattr--lsattr)
10. [SELinux & AppArmor Basics](#10-selinux--apparmor-basics)
11. [Capabilities](#11-capabilities)
12. [Common Secure Permission Values](#12-common-secure-permission-values)
13. [Debugging "Permission denied"](#13-debugging-permission-denied)
14. [Security Audit One-Liners](#14-security-audit-one-liners)
15. [Interview Questions](#15-interview-questions)
16. [Cheat Sheet](#16-cheat-sheet)

---

## 1. Reading Permissions

```bash
ls -l deploy.sh
-rwxr-x---  1  manoj  devops  2048  Sep 26 10:00  deploy.sh
```

```
 -   rwx   r-x   ---
 │    │     │     │
type owner group others
     (u)   (g)   (o)
```

| Char | Type |
|------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` / `c` | Block / character device |
| `s` / `p` | Socket / named pipe |

```bash
stat -c '%A %a %U:%G %n' deploy.sh      # -rwxr-x--- 750 manoj:devops deploy.sh
```

---

## 2. Permission Meaning: Files vs Directories

| Perm | On a **file** | On a **directory** |
|------|---------------|--------------------|
| `r` (4) | Read contents | List names (`ls`) |
| `w` (2) | Modify contents | Create / delete / rename files inside |
| `x` (1) | Execute as program | Enter (`cd`) and access files inside |

> **Key gotcha:** Deleting a file depends on the **directory's** `w` permission, not the file's.
> **Key gotcha 2:** To read `/a/b/c.txt` you need `x` on `/`, `/a`, `/a/b` **and** `r` on `c.txt`.

---

## 3. Numeric (Octal) Notation

| Value | Perms |
|-------|-------|
| 7 | `rwx` (4+2+1) |
| 6 | `rw-` (4+2) |
| 5 | `r-x` (4+1) |
| 4 | `r--` |
| 0 | `---` |

| Octal | Symbolic | Typical use |
|-------|----------|-------------|
| `755` | `rwxr-xr-x` | Scripts, binaries, public dirs |
| `750` | `rwxr-x---` | App dirs, team-only scripts |
| `700` | `rwx------` | `~/.ssh`, private dirs |
| `644` | `rw-r--r--` | Config files, web files |
| `640` | `rw-r-----` | Configs with secrets, logs |
| `600` | `rw-------` | Private keys, `.env`, `authorized_keys` |
| `400` | `r--------` | AWS `.pem` keys, read-only secrets |
| `775` | `rwxrwxr-x` | Shared team dirs |
| `777` | `rwxrwxrwx` | 🚨 Never in production |

---

## 4. chmod — Change Permissions

### Numeric

```bash
chmod 755 deploy.sh
chmod 600 ~/.ssh/id_ed25519
chmod 400 mykey.pem
chmod -R 750 /opt/app
```

### Symbolic — `[ugoa][+-=][rwxXst]`

```bash
chmod +x script.sh              # Execute for everyone
chmod u+x script.sh             # Execute for owner only
chmod g+w shared.txt            # Group can write
chmod o-rwx secret.conf         # Remove all from others
chmod go= secret.conf           # Group+others: nothing
chmod u=rwx,g=rx,o= app/        # Set exactly
chmod a+r readme.md             # All can read
chmod -R g+rX /srv/share        # Capital X: exec only for dirs ← safe recursive
chmod --reference=a.conf b.conf # Copy perms from another file
```

| Option / Symbol | What it does |
|-----------------|--------------|
| `-R` | Apply recursively to all contents |
| `-v` | Show every file being processed |
| `-c` | Report only files actually changed |
| `--reference=F` | Copy permissions from file F |
| `u` / `g` / `o` / `a` | Owner / group / others / all |
| `+` / `-` / `=` | Add / remove / set exactly |
| `X` | Execute only on dirs, executables |

**Safe recursive fix — dirs 755, files 644:**
```bash
find /var/www/html -type d -exec chmod 755 {} +
find /var/www/html -type f -exec chmod 644 {} +
```

---

## 5. chown & chgrp — Change Ownership

```bash
sudo chown manoj file.txt                 # Owner only
sudo chown manoj:devops file.txt          # Owner + group
sudo chown :devops file.txt               # Group only
sudo chown -R nginx:nginx /var/www/html   # Recursive
sudo chown -h manoj symlink               # The link itself, not target
sudo chown --reference=a b                # Copy ownership
sudo chown -R 1001:1001 /data             # Numeric (containers/volumes)
sudo chgrp -R devops /srv/project         # Change group only
```

| Option | What it does |
|--------|--------------|
| `-R` | Change ownership recursively for everything |
| `-h` | Change symlink itself, not target |
| `-v` / `-c` | Verbose / report only changes |
| `--reference=F` | Copy owner and group from F |
| `--from=OLD` | Change only if current owner matches |

> Only **root** can change a file's owner. A user can `chgrp` to a group they belong to.

> **Kubernetes/Docker tip:** "Permission denied" on volumes usually = container UID ≠ host file UID. Fix with `chown -R <uid>:<gid>` or `securityContext.fsGroup`.

---

## 6. Special Permissions: SUID, SGID, Sticky Bit

| Bit | Octal | On files | On directories | Shows as |
|-----|-------|----------|----------------|----------|
| **SUID** | 4000 | Runs as **file owner** | — | `rws` in user (`S` if no x) |
| **SGID** | 2000 | Runs as **file group** | New files **inherit dir's group** | `rws` in group |
| **Sticky** | 1000 | — | Only owner can delete their files | `rwt` in others |

```bash
ls -l /usr/bin/passwd       # -rwsr-xr-x  → SUID (edits /etc/shadow as root)
ls -ld /tmp                 # drwxrwxrwt  → sticky
chmod u+s /usr/local/bin/tool     # Set SUID (⚠️ rarely justified)
chmod 4755 tool
chmod g+s /srv/team               # SGID on shared dir ← very useful
chmod 2775 /srv/team
chmod +t /srv/upload              # Sticky bit
chmod 1777 /srv/upload
chmod u-s,g-s file                # Remove
```

**Perfect shared team directory**
```bash
sudo mkdir /srv/devops
sudo chown root:devops /srv/devops
sudo chmod 2770 /srv/devops        # SGID → all files owned by group devops
```

---

## 7. umask — Default Permissions

umask **removes** permissions from the base (files `666`, dirs `777`).

| umask | New file | New dir | Use |
|-------|----------|---------|-----|
| `022` | 644 | 755 | Default for most distros |
| `002` | 664 | 775 | Collaborative/group work |
| `027` | 640 | 750 | Hardened servers (CIS) |
| `077` | 600 | 700 | Highly private |

```bash
umask                  # 0022
umask -S               # u=rwx,g=rx,o=rx
umask 027              # Current session only
```

| Option | What it does |
|--------|--------------|
| `-S` | Show umask in symbolic form |

**Persistent:** `/etc/login.defs` (`UMASK`), `/etc/profile`, `~/.bashrc`, or systemd `UMask=0027`.

---

## 8. ACLs — Fine-Grained Access

When owner/group/others isn't enough (e.g., give one extra user read access).

```bash
getfacl /srv/reports                           # View ACLs
setfacl -m u:auditor:rx /srv/reports           # Give one user r-x
setfacl -m g:qa:rwx /srv/reports               # Give a group rwx
setfacl -R -m u:jenkins:rwX /opt/app           # Recursive
setfacl -d -m g:devops:rwx /srv/share          # DEFAULT ACL: new files inherit
setfacl -x u:auditor /srv/reports              # Remove one entry
setfacl -b /srv/reports                        # Remove all ACLs
getfacl dir1 | setfacl --set-file=- dir2       # Copy ACL
ls -l                                          # "+" at end = ACL present: drwxr-x---+
```

| Option | What it does |
|--------|--------------|
| `-m` | Modify or add an ACL entry |
| `-x` | Remove a specific ACL entry |
| `-b` | Remove all extended ACL entries |
| `-d` | Set default ACL for new files |
| `-R` | Apply ACL recursively to contents |
| `-k` | Remove only the default ACL |
| `--set-file` | Apply ACL entries from file |

> **ACL mask** limits max effective perms for named users/groups — check `mask::` in `getfacl` if an ACL seems ignored.

---

## 9. File Attributes — `chattr` / `lsattr`

Filesystem-level flags that even **root** must remove first.

```bash
sudo chattr +i /etc/resolv.conf     # Immutable: no edit/delete/rename (even root)
sudo chattr -i /etc/resolv.conf     # Remove immutable
sudo chattr +a /var/log/audit.log   # Append-only (tamper-resistant logs)
lsattr /etc/resolv.conf             # ----i---------e-- /etc/resolv.conf
lsattr -d /etc                      # Attributes of directory itself
```

| Attribute | What it does |
|-----------|--------------|
| `+i` | Immutable, nobody can modify/delete |
| `+a` | Append only, no overwrite/delete |
| `-R` (lsattr/chattr) | Apply or list recursively |

> "Operation not permitted" as **root**? → check `lsattr`.

---

## 10. SELinux & AppArmor Basics

**SELinux (RHEL, CentOS, Amazon Linux, Fedora)** — mandatory access control on top of normal perms.

```bash
getenforce                            # Enforcing / Permissive / Disabled
sestatus                              # Detailed status
sudo setenforce 0                     # Permissive temporarily (debug only!)
ls -Z /var/www/html                   # Show SELinux context
ps -eZ | grep nginx                   # Process context
sudo restorecon -Rv /var/www/html     # Reset to default context ← most common fix
sudo chcon -t httpd_sys_content_t /data/web -R   # Change context temporarily
sudo semanage fcontext -a -t httpd_sys_content_t "/data/web(/.*)?"   # Permanent rule
sudo restorecon -Rv /data/web
sudo semanage port -a -t http_port_t -p tcp 8081  # Allow nginx on new port
getsebool -a | grep httpd
sudo setsebool -P httpd_can_network_connect on    # Nginx reverse proxy to backend
sudo ausearch -m avc -ts recent                    # Recent denials
sudo sealert -a /var/log/audit/audit.log           # Human-readable explanation
```

| Option | What it does |
|--------|--------------|
| `ls -Z` | Show SELinux security context labels |
| `restorecon -R` | Recursively restore default contexts |
| `restorecon -v` | Print each context change made |
| `setsebool -P` | Make boolean change permanent across reboots |
| `semanage fcontext -a` | Add persistent file context rule |
| `ausearch -m avc` | Search audit log for denials |

**AppArmor (Ubuntu, Debian, SUSE)**
```bash
sudo aa-status                        # Loaded profiles
sudo aa-complain /usr/sbin/nginx      # Log-only mode
sudo aa-enforce /usr/sbin/nginx       # Enforce
```

> Files have correct `chmod` but still denied on RHEL? → **SELinux**. Check `ausearch -m avc`. Don't disable SELinux — fix the context.

---

## 11. Capabilities

Give a binary a *slice* of root power instead of SUID.

```bash
getcap /usr/bin/ping                            # cap_net_raw=ep
sudo setcap 'cap_net_bind_service=+ep' /opt/app/server   # Bind port <1024 without root
sudo setcap -r /opt/app/server                  # Remove
getcap -r / 2>/dev/null                         # Audit all
```

In systemd: `AmbientCapabilities=CAP_NET_BIND_SERVICE`

---

## 12. Common Secure Permission Values

| Path | Perms | Owner |
|------|-------|-------|
| `/etc/passwd` | 644 | root:root |
| `/etc/shadow` | 000 or 640 | root:root / root:shadow |
| `/etc/sudoers` | 440 | root:root |
| `/etc/ssh/sshd_config` | 600 | root:root |
| `~/.ssh/` | 700 | user |
| `~/.ssh/authorized_keys` | 600 | user |
| `~/.ssh/id_ed25519` | 600 | user |
| `~/.ssh/id_ed25519.pub` | 644 | user |
| Home dir | 700 or 750 | user |
| `/tmp` | 1777 | root |
| Web root files | 644 (dirs 755) | nginx/www-data or deploy user |
| App `.env` / secrets | 600 or 640 | app user |
| Cron scripts | 700 / 750 | root |

---

## 13. Debugging "Permission denied"

Work through this order:

```bash
# 1. Who am I running as?
id
ps -o user,pid,cmd -p <PID>                 # For a service

# 2. Perms on the file AND every parent dir
namei -l /var/www/html/app/index.html       # ← best single command

# 3. ACLs?
getfacl /path/to/file

# 4. Immutable / append-only?
lsattr /path/to/file

# 5. SELinux / AppArmor?
ls -Z /path ; ausearch -m avc -ts recent
dmesg | grep -i apparmor

# 6. Mounted read-only or noexec?
findmnt -T /path                            # Look for ro, noexec
mount | grep noexec

# 7. Test as the service user
sudo -u nginx cat /var/www/html/index.html
sudo -u nginx test -r file && echo readable
```

| Error | Usual cause |
|-------|-------------|
| `bash: ./script.sh: Permission denied` | No `x` bit, or FS mounted `noexec` |
| `Permission denied (publickey)` | `~/.ssh` perms too open or wrong owner |
| `UNPROTECTED PRIVATE KEY FILE!` | Key must be `600` / `400` |
| Nginx 403 Forbidden | Parent dir lacks `x`, wrong owner, or SELinux |
| `Operation not permitted` as root | `chattr +i` or SELinux |
| Container can't write volume | UID mismatch |

---

## 14. Security Audit One-Liners

```bash
find / -xdev -type f -perm -4000 2>/dev/null         # SUID files
find / -xdev -type f -perm -2000 2>/dev/null         # SGID files
find / -xdev -type f -perm -0002 2>/dev/null         # World-writable files 🚨
find / -xdev -type d -perm -0002 ! -perm -1000 2>/dev/null   # World-writable dirs without sticky
find / -xdev \( -nouser -o -nogroup \) 2>/dev/null   # Orphaned files
find /home -name "authorized_keys" -perm /077        # Loose SSH key perms
stat -c '%a %n' /etc/passwd /etc/shadow /etc/sudoers # Critical file perms
```

---

## 15. Interview Questions

1. **What does 755 mean?** Owner rwx, group r-x, others r-x.
2. **`x` on a directory?** Allows `cd` into it and access files inside.
3. **Can you delete a file you have no write permission on?** Yes, if you have `w`+`x` on the directory (unless sticky bit set).
4. **What is SGID on a directory?** New files inherit the directory's group.
5. **Why sticky bit on `/tmp`?** Users can't delete each other's files.
6. **umask 027 — new file perms?** 640; dirs 750.
7. **Root can't delete a file — why?** Immutable attribute (`chattr +i`) or read-only mount.
8. **Nginx 403 with correct perms on RHEL?** SELinux context — `restorecon -Rv`.
9. **`chmod -R 755` vs `chmod -R u=rwX,go=rX`?** Latter only gives `x` to dirs/executables — safer.

---

## 16. Cheat Sheet

```bash
ls -l | stat -c '%a %U:%G' | namei -l
chmod 755 / 644 / 600 / 400 | chmod u+x,g-w,o= | chmod -R g+rX
chown -R user:group | chown :group | chgrp -R
chmod 4755 SUID | 2775 SGID dir | 1777 sticky
umask 022 / 027 / 077 | umask -S
getfacl | setfacl -m u:x:rx | setfacl -d -m g:y:rwx | setfacl -b
chattr +i / +a | lsattr
getenforce | ls -Z | restorecon -Rv | setsebool -P | ausearch -m avc
getcap | setcap cap_net_bind_service=+ep
find / -perm -4000 | -perm -0002 | -nouser
```
