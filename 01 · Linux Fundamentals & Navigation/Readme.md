# 01 · Linux Fundamentals & Navigation

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Move around any Linux box fast, understand where you are, who you are, and what the system is — the first 60 seconds on every server.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Shell Basics](#2-shell-basics)
3. [Where Am I? — Navigation](#3-where-am-i--navigation)
4. [Listing Files — `ls`](#4-listing-files--ls)
5. [Who Am I? — Identity & Session](#5-who-am-i--identity--session)
6. [What Is This System? — System Info](#6-what-is-this-system--system-info)
7. [Getting Help](#7-getting-help)
8. [Finding Commands & Files](#8-finding-commands--files)
9. [Shell History & Productivity](#9-shell-history--productivity)
10. [Environment Variables](#10-environment-variables)
11. [Redirection, Pipes & Operators](#11-redirection-pipes--operators)
12. [Aliases & Shell Config Files](#12-aliases--shell-config-files)
13. [Keyboard Shortcuts](#13-keyboard-shortcuts)
14. [Real-World: First 60 Seconds on a Server](#14-real-world-first-60-seconds-on-a-server)
15. [Interview Questions](#15-interview-questions)
16. [Cheat Sheet](#16-cheat-sheet)

---

## 1. Core Concepts

| Term | Simple Meaning |
|------|----------------|
| **Kernel** | Core of the OS; talks to hardware |
| **Shell** | Program that reads your commands (`bash`, `zsh`, `sh`) |
| **Terminal** | Window that runs the shell |
| **Distribution (distro)** | Kernel + tools bundled (RHEL, Ubuntu, Amazon Linux) |
| **Absolute path** | Starts from `/` → `/etc/nginx/nginx.conf` |
| **Relative path** | Starts from current dir → `../logs/app.log` |
| **`~`** | Your home directory (`/home/manoj`) |
| **`.` / `..`** | Current directory / parent directory |
| **Prompt `$` vs `#`** | `$` = normal user, `#` = root |

> **Everything in Linux is a file** — devices (`/dev/sda`), processes (`/proc/1234`), sockets, config.

---

## 2. Shell Basics

```bash
echo $SHELL            # Your default login shell
echo $0                # Shell running right now
cat /etc/shells        # All shells installed
chsh -s /bin/zsh       # Change default shell
type cd                # Is it builtin, alias, or file?
```

**Command anatomy**

```
command  -options  arguments
  ls       -lah      /var/log
```

| Style | Example | Note |
|-------|---------|------|
| Short option | `-l` | Single dash, single letter |
| Combined short | `-lah` | Same as `-l -a -h` |
| Long option | `--human-readable` | Double dash, full word |
| End of options | `--` | Treat rest as filenames |

---

## 3. Where Am I? — Navigation

### `pwd` — Print Working Directory

```bash
pwd          # /home/manoj
pwd -P       # Real path, symlinks resolved
```

| Option | What it does |
|--------|--------------|
| `-P` | Show physical path, resolve symlinks |
| `-L` | Show logical path (default) |

### `cd` — Change Directory

```bash
cd /var/log          # Absolute path
cd logs              # Relative path
cd ..                # Up one level
cd ../..             # Up two levels
cd ~   or   cd       # Home directory
cd -                 # Previous directory (toggle)
cd ~deploy           # Another user's home
```

### `pushd` / `popd` / `dirs` — Directory Stack

```bash
pushd /etc/nginx     # Go there, remember where you were
pushd /var/log/nginx # Stack another
dirs -v              # Show stack with numbers
popd                 # Return to previous dir
```

| Option | What it does |
|--------|--------------|
| `dirs -v` | List stack, one per line numbered |
| `dirs -c` | Clear the whole directory stack |
| `pushd +N` | Rotate stack to Nth entry |

> **SRE tip:** `pushd`/`popd` is great when jumping between a config dir and a log dir during an incident.

---

## 4. Listing Files — `ls`

```bash
ls                  # Basic list
ls -l               # Long format
ls -lah             # Long, hidden, human sizes  ← most used
ls -ltr             # Oldest first, newest at bottom ← log hunting
ls -lS              # Largest files first
ls -ld /etc         # Info about dir itself
ls -li              # Show inode numbers
ls -R /etc/nginx    # Recursive
ls -1               # One file per line (scripts)
ls --color=auto     # Colorized output
```

| Option | What it does |
|--------|--------------|
| `-l` | Long format: perms, owner, size, date |
| `-a` | Show hidden dotfiles too |
| `-A` | Hidden files, skip `.` and `..` |
| `-h` | Human-readable sizes (K, M, G) |
| `-t` | Sort by modification time, newest |
| `-r` | Reverse the sort order |
| `-S` | Sort by file size, largest |
| `-d` | Show directory itself, not contents |
| `-i` | Print inode number of file |
| `-R` | List subdirectories recursively |
| `-1` | One entry per output line |
| `-F` | Append `/` `*` `@` type indicators |
| `-n` | Numeric UID/GID instead of names |
| `-Z` | Show SELinux security context |
| `--time-style=long-iso` | Full date in ISO format |

### Reading `ls -l` output

```
-rw-r--r--  1  root  root  4096  Sep 26 10:15  nginx.conf
│└──┬───┘   │   │     │     │        │             │
│ perms    links owner group size  modified       name
└─ type: - file, d dir, l link, c char dev, b block dev, s socket, p pipe
```

---

## 5. Who Am I? — Identity & Session

```bash
whoami              # Current username
id                  # UID, GID, all groups
id deploy           # Another user's IDs
groups              # Groups you belong to
who                 # Who is logged in now
w                   # Who + what they're running + load
last -n 20          # Last 20 logins
lastb               # Failed logins (root only)
lastlog             # Last login per user
uptime              # Uptime + load average
tty                 # Your terminal device
```

| Command / Option | What it does |
|------------------|--------------|
| `id -u` | Print only the user ID |
| `id -g` | Print only primary group ID |
| `id -Gn` | All group names, space separated |
| `w -h` | Skip the header line |
| `last -n 20` | Show only last 20 entries |
| `last -x` | Include shutdown and runlevel changes |
| `last reboot` | Show system reboot history |
| `uptime -p` | Pretty format: "up 3 days" |
| `uptime -s` | Show exact boot start time |

> **Load average** `0.50, 0.80, 1.20` = 1, 5, 15 min averages. Compare against CPU count (`nproc`). Load > nproc = saturated.

---

## 6. What Is This System? — System Info

```bash
hostname                    # Host name
hostname -I                 # All IP addresses
hostnamectl                 # Hostname, OS, kernel, virtualization
hostnamectl set-hostname web01.prod
uname -a                    # All kernel info
uname -r                    # Kernel release only
cat /etc/os-release         # Distro name & version ← most reliable
lsb_release -a              # Distro info (Debian/Ubuntu)
arch                        # CPU architecture
nproc                       # Number of CPUs
lscpu                       # CPU details
free -h                     # Memory usage
timedatectl                 # Time, timezone, NTP sync
date                        # Current date/time
date -u                     # UTC time
cal                         # Calendar
```

| Command / Option | What it does |
|------------------|--------------|
| `uname -a` | Print all kernel system info |
| `uname -r` | Kernel release version only |
| `uname -m` | Machine hardware name (x86_64) |
| `hostname -I` | Show all assigned IP addresses |
| `hostname -f` | Show fully qualified domain name |
| `free -h` | Memory in human-readable units |
| `free -m` | Memory shown in megabytes |
| `date +%F` | Date as YYYY-MM-DD format |
| `date +%s` | Unix epoch seconds since 1970 |
| `date -d @1700000000` | Convert epoch to readable date |
| `timedatectl set-timezone UTC` | Change the system time zone |

```bash
# Common in scripts / backups
BACKUP=db_$(date +%F_%H%M%S).sql      # db_2026-09-26_101530.sql
```

---

## 7. Getting Help

```bash
man ls                  # Full manual
man 5 passwd            # Section 5 (file formats)
man -k disk             # Search man pages by keyword
apropos network         # Same as man -k
ls --help               # Quick usage summary
help cd                 # Help for shell builtins
info coreutils          # GNU info docs
tldr tar                # Community examples (install tldr)
whatis ls               # One-line description
```

| Man Section | Contains |
|-------------|----------|
| 1 | User commands |
| 5 | Config file formats (`/etc/passwd`) |
| 8 | Admin commands (`mount`, `iptables`) |

**Inside `man`:** `/word` search · `n` next match · `q` quit · `G` end · `g` top

---

## 8. Finding Commands & Files

```bash
which nginx             # Path of executable in $PATH
which -a python3        # All matches in $PATH
type -a ls              # Alias/builtin/file info
whereis nginx           # Binary, source, man page
command -v docker       # POSIX way — use in scripts
locate nginx.conf       # Fast search via DB
sudo updatedb           # Refresh locate database
```

```bash
# Script-safe check if a tool exists
if ! command -v jq >/dev/null 2>&1; then
  echo "jq not installed"; exit 1
fi
```

> `find` is covered in depth in **02 · File & Directory Operations**.

---

## 9. Shell History & Productivity

```bash
history                 # Show command history
history 20              # Last 20 commands
!!                      # Re-run last command
sudo !!                 # Re-run last command with sudo ← classic
!123                    # Run history line 123
!ssh                    # Last command starting with "ssh"
!$                      # Last argument of previous command
!*                      # All args of previous command
^old^new                # Re-run last cmd replacing text
history -c              # Clear history (session)
Ctrl+R                  # Reverse search history ← most used
```

| Variable | What it does |
|----------|--------------|
| `HISTSIZE=10000` | Commands kept in memory session |
| `HISTFILESIZE=20000` | Lines kept in history file |
| `HISTTIMEFORMAT="%F %T "` | Add timestamps to history output |
| `HISTCONTROL=ignoreboth` | Skip duplicates and space-prefixed commands |

> **Security tip:** Start a command with a space (with `ignoreboth`) to keep secrets out of history.

---

## 10. Environment Variables

```bash
env                     # All environment variables
printenv PATH           # One variable
echo $HOME $USER $PATH
export APP_ENV=prod     # Set for this shell + children
unset APP_ENV           # Remove variable
export PATH=$PATH:/opt/tools/bin   # Append to PATH
env -i bash             # Start shell with empty env
APP_ENV=dev ./run.sh    # Set only for one command
```

| Variable | Meaning |
|----------|---------|
| `PATH` | Dirs searched for commands |
| `HOME` | User's home directory |
| `USER` | Current username |
| `SHELL` | Default login shell |
| `PWD` / `OLDPWD` | Current / previous directory |
| `PS1` | Prompt format |
| `LANG` | Locale |
| `EDITOR` | Default editor (`vim`) |
| `?` | Exit code of last command (`echo $?`) |
| `$$` | PID of current shell |

---

## 11. Redirection, Pipes & Operators

| Syntax | What it does |
|--------|--------------|
| `>` | Write stdout to file, overwrite |
| `>>` | Append stdout to end of file |
| `2>` | Redirect stderr (errors) to file |
| `2>&1` | Send stderr to same as stdout |
| `&>` | Redirect both stdout and stderr |
| `<` | Read stdin from a file |
| `<<EOF` | Here-doc: multi-line input block |
| `\|` | Pipe output into next command |
| `\| tee file` | Show on screen and save file |
| `/dev/null` | Black hole — discards everything |
| `;` | Run next command regardless |
| `&&` | Run next only if success |
| `\|\|` | Run next only if failure |
| `&` | Run command in background |
| `$(cmd)` | Substitute command output inline |

```bash
./deploy.sh > deploy.log 2>&1          # Capture everything
cron_job.sh >/dev/null 2>&1            # Silence completely
make build && make deploy || echo "FAILED"
systemctl status nginx | tee status.txt
cat <<EOF > /etc/motd
Production server - authorized access only
EOF
```

**Exit codes:** `0` = success · non-zero = failure. `echo $?` right after a command.

---

## 12. Aliases & Shell Config Files

```bash
alias ll='ls -lah'
alias k='kubectl'
alias gs='git status'
alias                   # List all aliases
unalias ll              # Remove alias
\ls                     # Bypass alias once
```

| File | When it loads |
|------|---------------|
| `/etc/profile` | Login shell, all users |
| `~/.bash_profile` / `~/.profile` | Login shell, your user |
| `~/.bashrc` | Every interactive non-login shell |
| `/etc/bashrc` / `/etc/bash.bashrc` | Interactive, all users |
| `~/.bash_logout` | When login shell exits |

```bash
source ~/.bashrc        # Reload config without logging out
. ~/.bashrc             # Same thing
```

---

## 13. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` / `Tab Tab` | Auto-complete / show options |
| `Ctrl+C` | Kill running command |
| `Ctrl+Z` | Suspend (then `bg`/`fg`) |
| `Ctrl+D` | Exit shell / end input |
| `Ctrl+L` | Clear screen (`clear`) |
| `Ctrl+A` / `Ctrl+E` | Start / end of line |
| `Ctrl+U` / `Ctrl+K` | Cut to start / to end |
| `Ctrl+W` | Delete previous word |
| `Ctrl+Y` | Paste cut text |
| `Ctrl+R` | Search history |
| `Alt+.` | Insert last argument |

---

## 14. Real-World: First 60 Seconds on a Server

```bash
whoami && hostname && hostname -I       # Who/where am I? (right box?)
cat /etc/os-release | head -3           # Which distro?
uptime                                  # Recently rebooted? Load?
w                                       # Anyone else logged in?
nproc && free -h                        # CPU count, memory pressure
df -h                                   # Disk full?
last -x | head                          # Recent reboots/shutdowns
dmesg -T | tail -20                     # Kernel errors (OOM, disk)
systemctl --failed                      # Failed services
journalctl -p err -S "-1h" --no-pager   # Errors in last hour
```

> Always confirm **hostname + environment** before running anything destructive. Many prod outages are "right command, wrong server".

---

## 15. Interview Questions

1. **Absolute vs relative path?** Absolute starts at `/`; relative starts from `pwd`.
2. **What does `cd -` do?** Returns to the previous directory.
3. **`.bashrc` vs `.bash_profile`?** Profile = login shells; bashrc = interactive non-login shells.
4. **How to see load average and what is "high"?** `uptime`; high when consistently > `nproc`.
5. **What is `2>&1`?** Redirect stderr to wherever stdout goes.
6. **`which` vs `command -v`?** `command -v` is POSIX, builtin, reliable in scripts.
7. **How do you re-run last command with sudo?** `sudo !!`
8. **How to find exit status?** `echo $?` — 0 is success.

---

## 16. Cheat Sheet

```bash
pwd | cd - | cd ~ | pushd/popd
ls -lah | ls -ltr | ls -lS | ls -ld
whoami | id | w | last | uptime
hostnamectl | uname -r | cat /etc/os-release | nproc | free -h
man -k | --help | tldr | which | command -v | type
history | Ctrl+R | !! | sudo !! | !$
export VAR=x | env | echo $?
> >> 2>&1 &> | tee && || ;
alias ll='ls -lah' | source ~/.bashrc
```
