# 09 · Sudo & Privilege Management

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Grant just enough root power, safely and auditably — the backbone of least privilege, change control and compliance (SOX / PCI / SOC2).

---

## Table of Contents

1. [Why sudo (Not Root Login)](#1-why-sudo-not-root-login)
2. [Using sudo — Everyday Commands](#2-using-sudo--everyday-commands)
3. [Granting sudo via Groups](#3-granting-sudo-via-groups)
4. [The sudoers File — Always Use `visudo`](#4-the-sudoers-file--always-use-visudo)
5. [sudoers Syntax Explained](#5-sudoers-syntax-explained)
6. [Aliases — Clean, Scalable Rules](#6-aliases--clean-scalable-rules)
7. [Tags: NOPASSWD, NOEXEC, SETENV](#7-tags-nopasswd-noexec-setenv)
8. [Defaults — Hardening & Logging](#8-defaults--hardening--logging)
9. [Real-World sudoers Examples](#9-real-world-sudoers-examples)
10. [Dangerous Rules (Privilege Escalation Traps)](#10-dangerous-rules-privilege-escalation-traps)
11. [Auditing sudo Usage](#11-auditing-sudo-usage)
12. [su vs sudo vs sudo -i vs sudo -s](#12-su-vs-sudo-vs-sudo--i-vs-sudo--s)
13. [Other Privilege Tools: pkexec, doas, runuser, polkit](#13-other-privilege-tools)
14. [Disabling Root Login](#14-disabling-root-login)
15. [Troubleshooting](#15-troubleshooting)
16. [Interview Questions](#16-interview-questions)
17. [Cheat Sheet](#17-cheat-sheet)

---

## 1. Why sudo (Not Root Login)

| Direct root login | sudo |
|-------------------|------|
| Shared password | Each user uses own password |
| No accountability — "root did it" | Every command logged with real username |
| All-or-nothing | Per-command, per-host granularity |
| Password rotation pain | Revoke one user without affecting others |

---

## 2. Using sudo — Everyday Commands

```bash
sudo systemctl restart nginx         # Run one command as root
sudo -u postgres psql                # Run as another user
sudo -u appsvc -g appgrp ./run.sh    # Specific user + group
sudo -i                              # Root login shell (root's env)
sudo -s                              # Root shell, keep your env
sudo -l                              # What am I allowed to run? ← check first
sudo -l -U manoj                     # What can manoj run? (root)
sudo -v                              # Refresh cached credentials
sudo -k                              # Forget cached credentials now
sudo -K                              # Remove timestamp entirely
sudo -E ./deploy.sh                  # Preserve environment (if allowed)
sudo -n true && echo "no password needed"   # Non-interactive test (scripts/Ansible)
sudo -b long_task                    # Run in background
sudoedit /etc/nginx/nginx.conf       # Safe edit (= sudo -e)
sudo !!                              # Re-run last command with sudo
sudo -H pip install x                # Set HOME to target user's home
```

| Option | What it does |
|--------|--------------|
| `-u USER` | Run command as this user |
| `-g GROUP` | Run command with this group |
| `-i` | Root login shell with root env |
| `-s` | Shell as root, keeps your env |
| `-l` | List commands you're allowed to run |
| `-U USER` | With -l, list another user's rights |
| `-v` | Validate, extend credential cache timeout |
| `-k` | Invalidate cached credentials, ask again |
| `-n` | Non-interactive, fail if password needed |
| `-E` | Preserve current environment variables |
| `-H` | Set HOME to target user's |
| `-e` | Edit files safely (sudoedit) |
| `-b` | Run the command in background |
| `-S` | Read password from stdin |

---

## 3. Granting sudo via Groups

| Distro | Admin group | Default rule |
|--------|-------------|--------------|
| RHEL / CentOS / Amazon Linux / Fedora | `wheel` | `%wheel ALL=(ALL) ALL` |
| Ubuntu / Debian | `sudo` | `%sudo ALL=(ALL:ALL) ALL` |

```bash
sudo usermod -aG wheel manoj      # RHEL
sudo usermod -aG sudo manoj       # Ubuntu
getent group wheel sudo           # Who has full sudo
```

> In production, prefer **drop-in files** in `/etc/sudoers.d/` over giving everyone `wheel`.

---

## 4. The sudoers File — Always Use `visudo`

`visudo` locks the file and **validates syntax before saving**. A syntax error in sudoers = nobody can sudo.

```bash
sudo visudo                                  # Edit /etc/sudoers
sudo visudo -f /etc/sudoers.d/devops         # Edit a drop-in file ← best practice
sudo visudo -c                               # Check all sudoers syntax
sudo visudo -cf /etc/sudoers.d/devops        # Check one file
EDITOR=vim sudo -E visudo                    # Choose editor
```

| Option | What it does |
|--------|--------------|
| `-c` | Check syntax only, don't edit |
| `-f FILE` | Edit or check this specific file |
| `-s` | Strict mode, stricter syntax checks |

**Drop-in rules (`/etc/sudoers.d/`)**
- Included by `@includedir /etc/sudoers.d` (or `#includedir`) at the end of `/etc/sudoers`.
- Filenames must **not** contain `.` or end in `~` (e.g., `devops.conf` is **ignored**!).
- Perms must be `0440`, owner `root:root`.

```bash
sudo install -m 0440 -o root -g root devops /etc/sudoers.d/devops
```

---

## 5. sudoers Syntax Explained

```
WHO     WHERE = (AS_USER:AS_GROUP)  TAGS:  COMMANDS
manoj   ALL   = (ALL:ALL)                  ALL
%devops ALL   = (root)              NOPASSWD: /usr/bin/systemctl restart nginx
```

| Field | Meaning |
|-------|---------|
| `manoj` / `%devops` | User, or `%` for group |
| First `ALL` | On which hosts (for shared sudoers) |
| `(ALL:ALL)` | Can run as any user : any group |
| `NOPASSWD:` | Tag — no password prompt |
| Last `ALL` / path | Allowed commands — **always full path** |

**Rules**
- Use **absolute paths** (`/usr/bin/systemctl`, find with `which`).
- **Last matching rule wins** — order matters.
- `!` negates: `!/usr/bin/su`.
- Arguments: `/usr/bin/systemctl restart nginx` allows *only* those args; `""` means no args allowed.
- Wildcards `*` in arguments are risky (see §10).

---

## 6. Aliases — Clean, Scalable Rules

```sudoers
## User aliases
User_Alias   SRE      = manoj, priya, alex
User_Alias   DEVS     = %developers

## Host aliases
Host_Alias   WEBSERVERS = web01, web02, 10.0.1.0/24

## Run-as aliases
Runas_Alias  APPUSERS = appsvc, tomcat

## Command aliases
Cmnd_Alias   SVC_NGINX = /usr/bin/systemctl start nginx, \
                         /usr/bin/systemctl stop nginx, \
                         /usr/bin/systemctl restart nginx, \
                         /usr/bin/systemctl reload nginx, \
                         /usr/bin/systemctl status nginx
Cmnd_Alias   LOGS      = /usr/bin/journalctl, /usr/bin/tail -f /var/log/*
Cmnd_Alias   PKG       = /usr/bin/dnf, /usr/bin/yum, /usr/bin/apt
Cmnd_Alias   SHELLS    = /bin/sh, /bin/bash, /usr/bin/su

## Rules
SRE    ALL        = (ALL) ALL, !SHELLS
DEVS   WEBSERVERS = (root) NOPASSWD: SVC_NGINX, LOGS
DEVS   ALL        = (APPUSERS) NOPASSWD: ALL
```

| Alias type | Groups together |
|------------|-----------------|
| `User_Alias` | Users or groups |
| `Host_Alias` | Hosts, IPs, networks |
| `Runas_Alias` | Target users to run as |
| `Cmnd_Alias` | Commands with full paths |

> Alias names must be **UPPERCASE**.

---

## 7. Tags: NOPASSWD, NOEXEC, SETENV

| Tag | What it does |
|-----|--------------|
| `NOPASSWD:` | Run without asking for password |
| `PASSWD:` | Require password (default) |
| `NOEXEC:` | Block command from spawning other programs |
| `SETENV:` | Allow user to keep environment |
| `LOG_INPUT:` / `LOG_OUTPUT:` | Record session keystrokes / output |

```sudoers
jenkins ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp
manoj   ALL=(root) NOEXEC: /usr/bin/less /var/log/*     # can't :!sh out of less
```

---

## 8. Defaults — Hardening & Logging

```sudoers
Defaults    use_pty                              # Run in pseudo-terminal (anti-hijack)
Defaults    logfile="/var/log/sudo.log"          # Dedicated sudo log
Defaults    log_input, log_output                # Full session recording
Defaults    iolog_dir="/var/log/sudo-io/%{user}"
Defaults    timestamp_timeout=5                  # Re-ask password after 5 min (0 = always)
Defaults    passwd_tries=3
Defaults    badpass_message="Wrong password - attempt logged"
Defaults    env_reset                            # Clean environment (default)
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Defaults    requiretty                           # Only from real terminal (breaks some automation)
Defaults    lecture="always"
Defaults:jenkins  !requiretty                    # Per-user override
Defaults:%devops  timestamp_timeout=0
```

| Default | What it does |
|---------|--------------|
| `use_pty` | Run commands inside a pseudo-terminal |
| `logfile=` | Write sudo events to this file |
| `log_input,log_output` | Record full input/output of sessions |
| `timestamp_timeout=N` | Minutes before password asked again |
| `env_reset` | Start with a clean safe environment |
| `secure_path=` | Trusted PATH used for sudo commands |
| `requiretty` | Only allow sudo from real TTY |
| `passwd_tries=N` | Allowed password attempts before failure |

```bash
sudoreplay -l                     # List recorded sessions
sudoreplay <TSID>                 # Replay a session (audit)
```

---

## 9. Real-World sudoers Examples

**1. SRE team — full sudo, audited, no raw shells**
```sudoers
# /etc/sudoers.d/sre
%sre ALL=(ALL) ALL
```

**2. CI/CD deploy user — only restart its app, no password**
```sudoers
# /etc/sudoers.d/deploy
deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp, /usr/bin/systemctl status myapp
```

**3. Developers — read logs & manage one service on web hosts**
```sudoers
# /etc/sudoers.d/developers
Cmnd_Alias APP = /usr/bin/systemctl * myapp, /usr/bin/journalctl -u myapp *
%developers WEBSERVERS=(root) NOPASSWD: APP
```

**4. DBA — act as postgres only**
```sudoers
%dba ALL=(postgres) NOPASSWD: ALL
# use: sudo -u postgres psql
```

**5. Ansible automation account**
```sudoers
ansible ALL=(ALL) NOPASSWD: ALL
Defaults:ansible !requiretty
```
> Restrict the ansible account by SSH key + source IP (`from=` in `authorized_keys`) since sudo is wide open.

**6. Break-glass / emergency account** — full sudo, password in vault, alerts on every use.

**7. Time-bound access**
```bash
echo "vendor1 ALL=(root) /usr/bin/systemctl restart app" | sudo tee /etc/sudoers.d/vendor1
sudo chmod 440 /etc/sudoers.d/vendor1 && sudo visudo -c
echo "rm -f /etc/sudoers.d/vendor1" | at now + 4 hours      # Auto-revoke
```

---

## 10. Dangerous Rules (Privilege Escalation Traps)

Many "restricted" rules are actually **full root**. Check GTFOBins before granting.

| Rule allows | Escape to root |
|-------------|----------------|
| `vim`, `vi`, `less`, `more`, `man` | `:!/bin/bash` |
| `find` | `sudo find . -exec /bin/sh \;` |
| `awk` | `sudo awk 'BEGIN{system("/bin/sh")}'` |
| `tar` | `--checkpoint-action=exec=/bin/sh` |
| `cp`, `tee`, `dd` | Overwrite `/etc/sudoers` or `/etc/shadow` |
| `chmod`, `chown` | Change perms on `/etc/shadow` |
| `python`, `perl`, `ruby`, `node` | `os.system("/bin/sh")` |
| `bash script.sh` (user-writable) | Edit the script |
| `systemctl *` | `systemctl edit` → ExecStart root shell |
| `docker` | `docker run -v /:/host ...` |
| `/usr/bin/*` wildcards | Anything |
| `ALL, !/bin/bash` | Blacklists are bypassable (copy bash elsewhere) |

**Safer patterns**
- Allow exact commands + exact args.
- Use `sudoedit` for file edits (runs editor as **you**).
- Use `NOEXEC:` for pagers.
- Scripts in sudoers → owned by root, mode `0755`, dir not writable by user.

---

## 11. Auditing sudo Usage

```bash
# RHEL / Amazon Linux
sudo grep sudo /var/log/secure | tail
# Ubuntu / Debian
sudo grep sudo /var/log/auth.log | tail
# systemd journal (all distros)
journalctl _COMM=sudo --since today
journalctl _COMM=sudo | grep "COMMAND="
# Failed attempts
sudo grep "incorrect password\|NOT in sudoers" /var/log/secure /var/log/auth.log 2>/dev/null
# Who can sudo
getent group wheel sudo
sudo grep -rv '^#' /etc/sudoers /etc/sudoers.d/ | grep -v '^\s*$'
sudo -l -U username
# auditd rule for sudoers changes
sudo auditctl -w /etc/sudoers -p wa -k sudoers_change
sudo auditctl -w /etc/sudoers.d/ -p wa -k sudoers_change
sudo ausearch -k sudoers_change
```

Typical log line:
```
Sep 26 10:15:02 web01 sudo: manoj : TTY=pts/0 ; PWD=/home/manoj ; USER=root ; COMMAND=/usr/bin/systemctl restart nginx
```

---

## 12. su vs sudo vs sudo -i vs sudo -s

| Command | Password needed | Environment | Logged per command |
|---------|-----------------|-------------|--------------------|
| `su` | Target's (root) password | Keeps yours | ❌ |
| `su -` | Root's password | Root's full login env | ❌ |
| `sudo cmd` | **Your** password | Reset, secure | ✅ |
| `sudo -i` | Your password | Root's login env | Only the `-i` |
| `sudo -s` | Your password | Your env, root shell | Only the `-s` |
| `sudo su -` | Your password | Root's login env | Only `su` (avoid) |

> Prefer `sudo <cmd>` for traceability. Interactive root shells hide individual commands from logs (unless `log_output` is on).

---

## 13. Other Privilege Tools

| Tool | Use |
|------|-----|
| `runuser -u appsvc -- cmd` | Root running a command as another user (no PAM prompt) — used in init scripts |
| `pkexec cmd` | Polkit-based elevation (desktops) |
| `doas` | Minimal sudo alternative (OpenBSD, Alpine) |
| `polkit` rules | Allow non-root `systemctl` actions without sudo |
| `setpriv` / `capsh` | Fine-grained privilege dropping |

---

## 14. Disabling Root Login

```bash
sudo passwd -l root                            # Lock root password
# /etc/ssh/sshd_config
PermitRootLogin no
sudo sshd -t && sudo systemctl reload sshd
```

> Before locking root, **verify your own sudo works in a second session**.

---

## 15. Troubleshooting

| Error | Cause / Fix |
|-------|-------------|
| `user is not in the sudoers file. This incident will be reported.` | Add to `wheel`/`sudo` or a drop-in; re-login |
| `sudo: parse error in /etc/sudoers near line N` | Broken syntax → fix via `pkexec visudo` or root console/single-user mode |
| `sudo: /etc/sudoers.d/x is world writable` | `chmod 440` |
| Drop-in file ignored | Filename contains `.` (e.g., `x.conf`) or ends in `~` |
| `sudo: a terminal is required to read the password` | Script needs `NOPASSWD` or `-S`, or `requiretty` set |
| `sudo: unable to resolve host xyz` | Hostname missing in `/etc/hosts` |
| `command not found` with sudo only | Binary not in `secure_path` → use full path |
| Group change not effective | Log out/in, or `newgrp` |
| Cloud: locked out of sudo | AWS SSM / EC2 Serial Console / detach volume & fix on rescue instance |

---

## 16. Interview Questions

1. **Why edit sudoers with `visudo`?** Locks file and validates syntax — prevents lockout.
2. **Grant user permission to restart only nginx?** `user ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx`
3. **Why is `ALL, !/bin/bash` insecure?** Blacklists are bypassable; copy/rename shell or use other escapes.
4. **Where are sudo logs?** `/var/log/secure` (RHEL), `/var/log/auth.log` (Ubuntu), `journalctl _COMM=sudo`.
5. **`sudo -i` vs `sudo -s`?** `-i` = root login env; `-s` = root shell with your env.
6. **Why are sudoers.d files ignored?** Names with dots or `~`, or wrong permissions.
7. **Why is sudo on `vim` dangerous?** `:!sh` spawns a root shell — use `sudoedit`.
8. **How to check a user's sudo rights?** `sudo -l -U user`.

---

## 17. Cheat Sheet

```bash
sudo cmd | sudo -u user | sudo -i | sudo -s | sudo -l | sudo -k | sudo -n | sudoedit
usermod -aG wheel|sudo user
visudo | visudo -f /etc/sudoers.d/team | visudo -c
USER HOST=(RUNAS:GROUP) NOPASSWD: /full/path args
%group  User_Alias  Host_Alias  Runas_Alias  Cmnd_Alias
Defaults use_pty, logfile=, log_output, timestamp_timeout=5, secure_path=
chmod 0440 /etc/sudoers.d/*  (no dots in names)
grep sudo /var/log/secure|auth.log | journalctl _COMM=sudo | sudoreplay
PermitRootLogin no | passwd -l root
```
