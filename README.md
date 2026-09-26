# Linux Command Reference — DevOps · SRE · Production Engineering


## Table of Contents

**Part 1 — Working in the Shell**
1. [Navigation & Listing](#1-navigation--listing)
2. [System Info & Identity](#2-system-info--identity)
3. [Help & Finding Commands](#3-help--finding-commands)
4. [History, Environment, Redirection & Aliases](#4-history-environment-redirection--aliases)
5. [Vim Editor](#5-vim-editor)

**Part 2 — Files & Data**

6. [Files & Directories (CRUD)](#6-files--directories-crud)
7. [Links & File Info](#7-links--file-info)
8. [Finding, Comparing & Verifying Files](#8-finding-comparing--verifying-files)
9. [Text Processing & Filters](#9-text-processing--filters)
10. [JSON & YAML](#10-json--yaml)
11. [Archiving & Compression](#11-archiving--compression)
12. [File Transfer & Sync](#12-file-transfer--sync)
13. [Important Paths (Filesystem Hierarchy)](#13-important-paths-filesystem-hierarchy)

**Part 3 — Access & Security**

14. [Users & Groups](#14-users--groups)
15. [Passwords, Aging & Lockout](#15-passwords-aging--lockout)
16. [Permissions & Ownership](#16-permissions--ownership)
17. [ACLs, Attributes, SELinux & Capabilities](#17-acls-attributes-selinux--capabilities)
18. [Sudo & Privilege Management](#18-sudo--privilege-management)
19. [SSH & Remote Access](#19-ssh--remote-access)
20. [SSH Server Hardening & Troubleshooting](#20-ssh-server-hardening--troubleshooting)
21. [Offboarding (Access Removal)](#21-offboarding-access-removal)

**Part 4 — Software & Services**

22. [Package Management — RHEL (dnf / yum / rpm)](#22-package-management--rhel-dnf--yum--rpm)
23. [Package Management — Ubuntu (apt / dpkg)](#23-package-management--ubuntu-apt--dpkg)
24. [Other Package Tools, Patching & Kernel](#24-other-package-tools-patching--kernel)
25. [Services (systemd)](#25-services-systemd)
26. [Scheduling & Boot](#26-scheduling--boot)
27. [Logs](#27-logs)

**Part 5 — Processes & Performance**

28. [Viewing Processes](#28-viewing-processes)
29. [Signals, Jobs & Priority](#29-signals-jobs--priority)
30. [Deep Process Inspection](#30-deep-process-inspection)
31. [Performance & Memory](#31-performance--memory)

**Part 6 — Networking**

32. [Interfaces & Routing](#32-interfaces--routing)
33. [Ports & Sockets](#33-ports--sockets)
34. [Connectivity Testing](#34-connectivity-testing)
35. [DNS](#35-dns)
36. [HTTP & TLS](#36-http--tls)
37. [Network Configuration](#37-network-configuration)
38. [Firewalls](#38-firewalls)
39. [Packet Capture, Bandwidth & Kernel Tuning](#39-packet-capture-bandwidth--kernel-tuning)

**Part 7 — Disk & Storage**

40. [Disk Usage](#40-disk-usage)
41. [Disks, Partitions & Filesystems](#41-disks-partitions--filesystems)
42. [Mounting & fstab](#42-mounting--fstab)
43. [LVM](#43-lvm)
44. [Growing Storage, Swap & Repair](#44-growing-storage-swap--repair)
45. [Disk Performance, RAID, Network Storage & Encryption](#45-disk-performance-raid-network-storage--encryption)
46. [Containers & Kubernetes Storage](#46-containers--kubernetes-storage)

**Part 8 — Quick Reference**

47. [Incident First-Response Commands](#47-incident-first-response-commands)

---

# Part 1 — Working in the Shell

## 1. Navigation & Listing

| Command | What it does |
|---------|--------------|
| `pwd` | Prints the directory you are currently in. |
| `pwd -P` | Prints the real physical path with all symlinks resolved. |
| `cd /var/log` | Moves to the given absolute path. |
| `cd ..` | Moves up one directory level. |
| `cd ~` / `cd` | Returns to your home directory. |
| `cd -` | Jumps back to the previous directory you were in. |
| `cd ~deploy` | Goes to another user's home directory. |
| `pushd /etc/nginx` | Changes directory and saves the old one on a stack. |
| `popd` | Returns to the last directory saved by `pushd`. |
| `dirs -v` | Lists the directory stack with numbers. |
| `ls` | Lists files in the current directory. |
| `ls -lah` | Long listing including hidden files with human-readable sizes. |
| `ls -ltr` | Lists by modification time with the newest file last (great for logs). |
| `ls -lS` | Lists files sorted by size, largest first. |
| `ls -ld /etc` | Shows details of the directory itself, not its contents. |
| `ls -li` | Shows the inode number of each file. |
| `ls -R` | Lists directories recursively. |
| `ls -1` | Prints one entry per line (useful in scripts). |
| `ls -n` | Shows numeric UID/GID instead of names. |
| `ls -Z` | Shows SELinux security context of files. |
| `tree -L 2 /etc` | Displays a directory tree two levels deep. |
| `tree -d` | Displays only directories in tree form. |

## 2. System Info & Identity

| Command | What it does |
|---------|--------------|
| `whoami` | Prints your current username. |
| `id` | Shows your UID, GID and all group memberships. |
| `id -Gn user` | Lists all group names a user belongs to. |
| `groups` | Lists the groups you belong to. |
| `who` | Shows who is currently logged in. |
| `w` | Shows logged-in users, what they run and system load. |
| `tty` | Prints the terminal device you are using. |
| `last -n 20` | Shows the last 20 logins. |
| `last reboot` | Shows the system reboot history. |
| `last -x` | Includes shutdowns and runlevel changes in login history. |
| `lastb` | Shows failed login attempts (root only). |
| `lastlog` | Shows the last login time of every user. |
| `uptime` | Shows how long the system has run and load averages. |
| `uptime -s` | Shows the exact time the system booted. |
| `hostname` | Prints the system hostname. |
| `hostname -I` | Prints all IP addresses assigned to the host. |
| `hostname -f` | Prints the fully qualified domain name. |
| `hostnamectl` | Shows hostname, OS, kernel and virtualization type. |
| `hostnamectl set-hostname web01` | Permanently changes the hostname. |
| `uname -a` | Prints all kernel and system information. |
| `uname -r` | Prints the running kernel version. |
| `uname -m` | Prints the CPU architecture (e.g., x86_64). |
| `cat /etc/os-release` | Shows the distribution name and version reliably. |
| `lsb_release -a` | Shows distribution info on Debian/Ubuntu. |
| `arch` | Prints the machine architecture. |
| `nproc` | Prints the number of CPU cores available. |
| `lscpu` | Shows detailed CPU information. |
| `free -h` | Shows memory and swap usage in human units. |
| `timedatectl` | Shows time, timezone and NTP sync status. |
| `timedatectl set-timezone UTC` | Changes the system timezone. |
| `date` | Prints the current date and time. |
| `date -u` | Prints the current time in UTC. |
| `date +%F_%H%M%S` | Prints a timestamp for filenames (e.g., backups). |
| `date +%s` | Prints Unix epoch seconds. |
| `date -d @1700000000` | Converts an epoch number to a readable date. |
| `cal` | Shows a calendar. |

## 3. Help & Finding Commands

| Command | What it does |
|---------|--------------|
| `man ls` | Opens the full manual page for a command. |
| `man 5 passwd` | Opens a specific manual section (5 = file formats). |
| `man -k disk` / `apropos disk` | Searches manual pages by keyword. |
| `man hier` | Describes the Linux filesystem hierarchy. |
| `ls --help` | Prints a quick usage summary for a command. |
| `help cd` | Shows help for shell built-in commands. |
| `info coreutils` | Opens GNU info documentation. |
| `whatis ls` | Prints a one-line description of a command. |
| `tldr tar` | Shows short community examples (needs `tldr`). |
| `which nginx` | Shows which executable in `$PATH` will run. |
| `which -a python3` | Shows every match in `$PATH`. |
| `type -a ls` | Tells whether a name is an alias, builtin or file. |
| `command -v docker` | POSIX-safe way to check if a command exists (use in scripts). |
| `whereis nginx` | Shows binary, source and man page locations. |
| `locate nginx.conf` | Finds files instantly using a prebuilt database. |
| `sudo updatedb` | Refreshes the database used by `locate`. |

## 4. History, Environment, Redirection & Aliases

### History

| Command | What it does |
|---------|--------------|
| `history` | Lists your command history. |
| `history 20` | Shows the last 20 commands. |
| `history -c` | Clears the current session history. |
| `Ctrl+R` | Searches backward through history interactively. |
| `!!` | Re-runs the previous command. |
| `sudo !!` | Re-runs the previous command with sudo. |
| `!123` | Runs history entry number 123. |
| `!ssh` | Runs the last command that started with "ssh". |
| `!$` | Inserts the last argument of the previous command. |
| `^old^new` | Re-runs the last command replacing "old" with "new". |
| `HISTTIMEFORMAT="%F %T "` | Adds timestamps to history output. |
| `HISTCONTROL=ignoreboth` | Skips duplicates and commands starting with a space. |

### Environment variables

| Command | What it does |
|---------|--------------|
| `env` / `printenv` | Lists all environment variables. |
| `printenv PATH` | Prints one variable's value. |
| `echo $HOME $USER $SHELL` | Prints common built-in variables. |
| `echo $?` | Prints the exit code of the last command (0 = success). |
| `echo $$` | Prints the PID of the current shell. |
| `export APP_ENV=prod` | Sets a variable for this shell and its children. |
| `export PATH=$PATH:/opt/bin` | Appends a directory to the search path. |
| `unset APP_ENV` | Removes a variable. |
| `APP_ENV=dev ./run.sh` | Sets a variable for one command only. |
| `env -i bash` | Starts a shell with an empty environment. |
| `echo $SHELL` / `echo $0` | Shows your default shell / the shell running now. |
| `cat /etc/shells` | Lists shells installed on the system. |
| `chsh -s /bin/zsh` | Changes your default login shell. |

### Redirection & operators

| Syntax | What it does |
|--------|--------------|
| `cmd > file` | Writes output to a file, overwriting it. |
| `cmd >> file` | Appends output to the end of a file. |
| `cmd 2> err.log` | Sends error output to a file. |
| `cmd > out.log 2>&1` | Sends both output and errors to the same file. |
| `cmd &> file` | Shorthand to redirect both output and errors. |
| `cmd >/dev/null 2>&1` | Discards all output (common in cron). |
| `cmd < file` | Feeds a file as input to a command. |
| `cat <<EOF > file` | Writes a multi-line block (here-doc) to a file. |
| `cmd1 \| cmd2` | Pipes output of one command into the next. |
| `cmd \| tee file` | Shows output on screen and saves it to a file. |
| `cmd1 ; cmd2` | Runs commands one after another regardless of result. |
| `cmd1 && cmd2` | Runs the second command only if the first succeeds. |
| `cmd1 \|\| cmd2` | Runs the second command only if the first fails. |
| `cmd &` | Runs a command in the background. |
| `$(cmd)` | Inserts a command's output inline. |

### Aliases & shell config

| Command | What it does |
|---------|--------------|
| `alias ll='ls -lah'` | Creates a shortcut for a longer command. |
| `alias` | Lists all defined aliases. |
| `unalias ll` | Removes an alias. |
| `\ls` | Runs the real command, bypassing an alias once. |
| `source ~/.bashrc` / `. ~/.bashrc` | Reloads shell config without logging out. |
| `~/.bashrc` | Runs for every interactive shell (aliases, prompt). |
| `~/.bash_profile` | Runs for login shells (env vars, PATH). |
| `/etc/profile`, `/etc/profile.d/` | System-wide login shell settings. |

### Keyboard shortcuts

| Keys | What it does |
|------|--------------|
| `Tab` | Auto-completes commands and paths. |
| `Ctrl+C` | Stops the running command. |
| `Ctrl+Z` | Suspends the running command. |
| `Ctrl+D` | Exits the shell or ends input. |
| `Ctrl+L` | Clears the screen. |
| `Ctrl+A` / `Ctrl+E` | Moves cursor to start / end of the line. |
| `Ctrl+U` / `Ctrl+K` | Cuts text before / after the cursor. |
| `Ctrl+W` | Deletes the previous word. |
| `Alt+.` | Inserts the last argument of the previous command. |

## 5. Vim Editor

### Open, save, quit

| Command | What it does |
|---------|--------------|
| `vim file` | Opens or creates a file. |
| `vim +25 file` | Opens a file at line 25. |
| `vim +/ERROR file` | Opens a file at the first match of "ERROR". |
| `vim -R file` / `view file` | Opens a file read-only. |
| `vim -O a b` | Opens two files side by side. |
| `vimdiff a b` / `vim -d a b` | Opens files in diff mode. |
| `vim -r file` | Recovers unsaved changes from a swap file. |
| `sudoedit /etc/hosts` | Safely edits a root-owned file with your own editor settings. |
| `vimtutor` | Starts the built-in 30-minute Vim tutorial. |
| `:w` | Saves the file. |
| `:q` | Quits (fails if there are unsaved changes). |
| `:wq` / `:x` / `ZZ` | Saves and quits. |
| `:q!` | Quits and discards all changes. |
| `:w newname` | Saves to a new file name. |
| `:w !sudo tee %` | Saves a file when you forgot to open it with sudo. |
| `:e!` | Reloads the file and discards unsaved changes. |

### Modes & inserting

| Key | What it does |
|-----|--------------|
| `Esc` | Returns to Normal mode. |
| `i` / `a` | Inserts before / after the cursor. |
| `I` / `A` | Inserts at start / end of the line. |
| `o` / `O` | Opens a new line below / above. |
| `R` | Enters Replace (overwrite) mode. |
| `r<char>` | Replaces one character. |
| `v` / `V` / `Ctrl+v` | Starts character / line / block selection. |

### Moving

| Key | What it does |
|-----|--------------|
| `h j k l` | Moves left, down, up, right. |
| `w` / `b` / `e` | Next word / previous word / end of word. |
| `0` / `^` / `$` | Start of line / first character / end of line. |
| `gg` / `G` | First line / last line. |
| `:42` / `42G` | Goes to line 42. |
| `Ctrl+f` / `Ctrl+b` | Page down / page up. |
| `Ctrl+d` / `Ctrl+u` | Half page down / up. |
| `%` | Jumps to the matching bracket. |
| `*` / `#` | Searches the word under the cursor forward / backward. |
| `Ctrl+o` | Jumps back to the previous location. |
| `zz` | Centers the screen on the cursor. |

### Editing

| Key | What it does |
|-----|--------------|
| `x` | Deletes the character under the cursor. |
| `dd` / `5dd` | Deletes (cuts) one / five lines. |
| `dw` / `diw` | Deletes to next word / the whole word. |
| `D` | Deletes to the end of the line. |
| `dG` | Deletes to the end of the file. |
| `ci"` | Changes the text inside quotes (edit config values). |
| `cw` / `cc` / `C` | Changes a word / whole line / rest of line. |
| `yy` / `5yy` | Copies one / five lines. |
| `p` / `P` | Pastes after / before the cursor. |
| `u` | Undoes the last change. |
| `Ctrl+r` | Redoes the last undone change. |
| `.` | Repeats the last change. |
| `J` | Joins the next line onto the current one. |
| `>>` / `<<` | Indents / unindents the line. |
| `gg=G` | Auto-indents the whole file. |
| `~` | Toggles the case of a character. |
| `Ctrl+a` / `Ctrl+x` | Increments / decrements a number. |
| `:10,20d` | Deletes lines 10 to 20. |
| `:10,20m 30` | Moves lines 10–20 after line 30. |
| `Ctrl+v` → `I# ` → `Esc` | Comments out a block of lines. |

### Search & replace

| Command | What it does |
|---------|--------------|
| `/pattern` / `?pattern` | Searches forward / backward. |
| `n` / `N` | Goes to next / previous match. |
| `:noh` | Clears search highlighting. |
| `:%s/old/new/g` | Replaces all occurrences in the file. |
| `:%s/old/new/gc` | Replaces all occurrences with confirmation. |
| `:10,20s/old/new/g` | Replaces only in lines 10–20. |
| `:%s#/usr/local#/opt#g` | Replaces paths using a different delimiter. |
| `:%s/\s\+$//e` | Removes trailing whitespace. |
| `:%s/\r//g` | Removes Windows `^M` characters. |
| `:g/^#/d` | Deletes all comment lines. |
| `:g/^$/d` | Deletes all empty lines. |
| `:v/pattern/d` | Keeps only lines that match the pattern. |

### Files, windows, settings, shell

| Command | What it does |
|---------|--------------|
| `:e file` | Opens another file. |
| `:sp` / `:vsp file` | Splits the window horizontally / vertically. |
| `Ctrl+w w` | Switches between split windows. |
| `:ls` / `:bn` / `:bp` | Lists / next / previous open buffers. |
| `]c` / `[c` | Jumps to next / previous difference in vimdiff. |
| `do` / `dp` | Pulls / pushes a diff change between files. |
| `:set nu` | Shows line numbers. |
| `:set paste` | Pastes without auto-indent breaking text. |
| `:set list` | Shows tabs and line endings. |
| `:set et ts=2 sw=2` | Uses 2 spaces for indentation (YAML). |
| `:set ff=unix` | Converts Windows line endings to Unix. |
| `:!cmd` | Runs a shell command from Vim. |
| `:r !date` | Inserts a command's output into the file. |
| `:%!jq .` | Pretty-formats the whole file as JSON. |
| `qa … q` / `@a` | Records a macro into `a` / plays it back. |
| `Ctrl+q` | Unfreezes Vim after an accidental `Ctrl+s`. |

---

# Part 2 — Files & Data

## 6. Files & Directories (CRUD)

### Create

| Command | What it does |
|---------|--------------|
| `touch app.log` | Creates an empty file or updates its timestamp. |
| `touch file{1..5}.txt` | Creates several files at once with brace expansion. |
| `touch -d "2026-01-01" f` | Sets a file's timestamp to a specific date. |
| `mkdir releases` | Creates a directory. |
| `mkdir -p /opt/app/{bin,conf,logs}` | Creates a full directory tree, including parents. |
| `mkdir -m 750 /opt/secure` | Creates a directory with specific permissions. |
| `echo "text" > file` | Writes text to a file, overwriting it. |
| `echo "text" >> file` | Appends text to a file. |
| `printf "a\nb\n" > file` | Writes formatted multi-line text to a file. |
| `cat <<'EOF' > app.conf` | Writes a multi-line block to a file without variable expansion. |
| `echo "x" \| sudo tee /etc/file` | Writes to a root-owned file (plain `sudo echo >` fails). |
| `sudo tee -a /etc/file` | Appends to a root-owned file. |
| `install -m 0755 -o root -g root tool /usr/local/bin/` | Copies a file and sets mode and owner in one step. |
| `install -d -m 0750 -o app -g app /var/lib/app` | Creates a directory with owner and mode set. |
| `mktemp` / `mktemp -d` | Creates a safe unique temp file / directory. |
| `trap 'rm -rf "$TMP"' EXIT` | Cleans up temp files automatically when a script exits. |

### Read / view

| Command | What it does |
|---------|--------------|
| `cat file` | Prints the whole file. |
| `cat -n file` | Prints the file with line numbers. |
| `cat -A file` | Reveals hidden characters like tabs and Windows `^M`. |
| `less file` | Opens a file in a scrollable, searchable pager. |
| `less +F file` | Opens a file in follow mode like `tail -f`. |
| `head -n 20 file` | Shows the first 20 lines. |
| `tail -n 50 file` | Shows the last 50 lines. |
| `tail -f app.log` | Follows a file live as new lines are written. |
| `tail -F app.log` | Follows a file even across log rotation. |
| `tail -n +10 file` | Prints from line 10 to the end. |
| `more file` | Older pager that scrolls forward only. |
| `tac file` | Prints a file in reverse line order. |
| `nl -ba file` | Prints a file with all lines numbered. |
| `wc -l file` | Counts lines in a file. |
| `wc -w` / `wc -c` | Counts words / bytes. |

### Copy, move, rename

| Command | What it does |
|---------|--------------|
| `cp a b` | Copies a file. |
| `cp app.conf{,.bak}` | Makes a quick backup copy before editing. |
| `cp -r src/ dest/` | Copies a directory recursively. |
| `cp -a /var/www /backup/` | Copies preserving permissions, owners, times and links. |
| `cp -p file dest` | Copies preserving mode, owner and timestamps. |
| `cp -i` / `cp -n` | Asks before overwriting / never overwrites. |
| `cp -u src/* dest/` | Copies only files that are newer. |
| `mv old new` | Renames a file or directory. |
| `mv file /dir/` | Moves a file to another directory. |
| `mv -i` / `mv -n` | Asks before overwriting / never overwrites. |
| `rename 's/\.log$/.old/' *.log` | Bulk-renames files with a pattern (Perl rename). |
| `for f in *.txt; do mv "$f" "${f%.txt}.md"; done` | Portable bulk rename using a shell loop. |

### Delete & empty

| Command | What it does |
|---------|--------------|
| `rm file` | Deletes a file. |
| `rm -i *.log` | Deletes files asking for confirmation each time. |
| `rm -I *` | Asks once before deleting many files. |
| `rm -r dir/` | Deletes a directory and its contents. |
| `rm -rf dir/` | ⚠️ Force-deletes a directory with no prompts. |
| `rm -rf "${DIR:?}"/*` | Safely deletes contents only if the variable is set. |
| `rm -- -weirdname` | Deletes a file whose name starts with a dash. |
| `rmdir dir` | Deletes an empty directory. |
| `shred -uz -n 3 file` | ⚠️ Overwrites a file several times then deletes it. |
| `> app.log` / `: > app.log` | Empties a file without deleting it. |
| `truncate -s 0 app.log` | Empties a live log file so space is freed immediately. |

### Wildcards

| Pattern | What it does |
|---------|--------------|
| `*` | Matches any number of characters. |
| `?` | Matches exactly one character. |
| `[0-9]` / `[abc]` | Matches one character from a set. |
| `{a,b}` | Expands to both values (e.g., `{dev,prod}`). |
| `{1..10}` | Expands to a number sequence. |

## 7. Links & File Info

| Command | What it does |
|---------|--------------|
| `ln -s target linkname` | Creates a symbolic (soft) link. |
| `ln -sfn /releases/v2 /opt/app/current` | Repoints a symlink atomically (zero-downtime deploys). |
| `ln original hardlink` | Creates a hard link to the same data (inode). |
| `readlink -f path` | Resolves a link to its final real path. |
| `find /etc -xtype l` | Finds broken symlinks. |
| `find / -samefile file` | Finds all hard links to a file. |
| `stat file` | Shows size, inode, permissions and all timestamps. |
| `stat -c '%a %U:%G %n' file` | Prints octal permissions and owner in a custom format. |
| `file backup.tgz` | Detects the real file type regardless of extension. |
| `file -i file` | Shows the MIME type of a file. |

## 8. Finding, Comparing & Verifying Files

### find

| Command | What it does |
|---------|--------------|
| `find /etc -name "nginx.conf"` | Finds files by exact name. |
| `find / -iname "*.pem" 2>/dev/null` | Finds files by name ignoring case, hiding errors. |
| `find . -type f` / `-type d` / `-type l` | Finds only files / directories / symlinks. |
| `find . -maxdepth 1 -type f` | Searches only the current directory level. |
| `find / -xdev -type f -size +500M` | Finds large files on one filesystem (disk-full hunting). |
| `find /var/log -mtime +30` | Finds files modified more than 30 days ago. |
| `find /tmp -mmin -10` | Finds files modified in the last 10 minutes. |
| `find . -newer marker` | Finds files newer than a reference file. |
| `find /data -empty` | Finds empty files and directories. |
| `find /home -user deploy` | Finds files owned by a user. |
| `find / -nouser -o -nogroup` | Finds orphaned files with no valid owner. |
| `find / -perm -4000 -type f` | Finds SUID binaries (security audit). |
| `find / -perm -0002 -type f` | Finds world-writable files. |
| `find . -name "*.log" -mtime +7 -delete` | ⚠️ Deletes old log files that match. |
| `find . -name "*.sh" -exec chmod +x {} \;` | Runs a command once per matched file. |
| `find . -name "*.log" -exec gzip {} +` | Runs a command on many files at once (faster). |
| `find . -path ./node_modules -prune -o -name "*.js" -print` | Excludes a directory from the search. |
| `find . -type f -print0 \| xargs -0 ls -l` | Passes filenames safely even with spaces. |

### xargs

| Command | What it does |
|---------|--------------|
| `xargs -0` | Reads null-separated input from `find -print0`. |
| `xargs -n1` | Runs the command with one argument at a time. |
| `xargs -I{} ssh {} uptime` | Substitutes each input line into the command. |
| `xargs -P 8` | Runs up to 8 commands in parallel. |
| `xargs -r` | Does nothing if there is no input. |
| `xargs -t` | Prints each command before running it. |

### Compare & verify

| Command | What it does |
|---------|--------------|
| `diff a b` | Shows line differences between two files. |
| `diff -u a b` | Shows differences in readable unified (git-style) format. |
| `diff -r dir1 dir2` | Compares two directories recursively. |
| `diff -y a b` | Shows two files side by side. |
| `diff -q a b` | Only reports whether files differ. |
| `diff <(ssh web1 cat f) <(ssh web2 cat f)` | Compares a config across two servers (drift check). |
| `cmp a b` | Compares two files byte by byte (binaries). |
| `comm -23 <(sort a) <(sort b)` | Shows lines that exist only in the first file. |
| `sha256sum file` | Prints a SHA-256 checksum of a file. |
| `sha256sum -c file.sha256` | Verifies files against saved checksums. |
| `md5sum file` | Prints an MD5 checksum (quick integrity check). |

## 9. Text Processing & Filters

### grep — search

| Command | What it does |
|---------|--------------|
| `grep "ERROR" app.log` | Prints lines containing a pattern. |
| `grep -i error` | Searches ignoring case. |
| `grep -v DEBUG` | Prints lines that do NOT match. |
| `grep -n` | Shows line numbers with matches. |
| `grep -c 500` | Counts matching lines. |
| `grep -w fail` | Matches whole words only. |
| `grep -x "text"` | Matches the whole line exactly. |
| `grep -o "user=[a-z]*"` | Prints only the matched part of each line. |
| `grep -r "pattern" /etc/app/` | Searches recursively through a directory. |
| `grep -rl "8080" /etc/` | Prints only the names of files that match. |
| `grep -L pattern *` | Prints names of files that do not match. |
| `grep -A 5 -B 2 "Exception"` | Shows 2 lines before and 5 after each match. |
| `grep -C 3 OOM` | Shows 3 lines of context around each match. |
| `grep -E "ERROR\|WARN\|FATAL"` | Searches multiple patterns with extended regex. |
| `grep -F "a.b*c"` | Searches for a literal string with no regex. |
| `grep -P '\d{3}'` | Uses Perl regex features like `\d` and `\s`. |
| `grep -f patterns.txt` | Reads patterns from a file. |
| `grep -m 1 started` | Stops after the first match. |
| `grep -q ready && echo OK` | Returns only an exit code (for scripts). |
| `grep -rn --include="*.yaml" "image:" .` | Searches only files matching a name pattern. |
| `grep -rn --exclude-dir=.git TODO .` | Skips a directory while searching. |
| `grep -Ev '^\s*(#\|$)' file` | Shows a config without comments and blank lines. |
| `tail -f log \| grep --line-buffered ERROR` | Filters a live log stream without delay. |
| `rg pattern` | Fast recursive search with ripgrep (respects `.gitignore`). |

### cut, sort, uniq, tr

| Command | What it does |
|---------|--------------|
| `cut -d: -f1 /etc/passwd` | Extracts the first colon-separated field. |
| `cut -d, -f2- data.csv` | Extracts field 2 to the end. |
| `cut -c1-10` | Extracts characters 1 to 10 of each line. |
| `sort file` | Sorts lines alphabetically. |
| `sort -n` / `sort -rn` | Sorts numerically / numerically largest first. |
| `sort -h` | Sorts human sizes like 1K, 2M, 3G. |
| `sort -k3 -n` | Sorts by the third column numerically. |
| `sort -t: -k3 -n /etc/passwd` | Sorts by a column with a custom separator. |
| `sort -u` | Sorts and removes duplicate lines. |
| `sort -V` | Sorts version numbers correctly (1.2 < 1.10). |
| `uniq` | Removes adjacent duplicate lines (sort first). |
| `uniq -c` | Counts how many times each line appears. |
| `uniq -d` / `uniq -u` | Prints only duplicated / only unique lines. |
| `sort \| uniq -c \| sort -rn \| head` | Finds the top most frequent values (classic log analysis). |
| `tr 'a-z' 'A-Z'` | Converts lowercase to uppercase. |
| `tr -d '\r'` | Removes Windows carriage-return characters. |
| `tr -s ' '` | Squeezes repeated spaces into one. |
| `tr ':' '\n'` | Replaces a character with newlines (e.g., split `$PATH`). |
| `tr -dc 'A-Za-z0-9' </dev/urandom \| head -c 20` | Generates a random password. |

### sed — stream editor

| Command | What it does |
|---------|--------------|
| `sed 's/old/new/' file` | Replaces the first match on each line. |
| `sed 's/old/new/g' file` | Replaces every match on each line. |
| `sed -i 's/8080/9090/g' file` | ⚠️ Edits the file in place. |
| `sed -i.bak 's/a/b/g' file` | Edits in place and keeps a `.bak` backup. |
| `sed 's#/usr/local#/opt#g'` | Uses `#` as delimiter for paths. |
| `sed -E 's/(v)[0-9.]+/\12.0/'` | Uses extended regex with capture groups. |
| `sed -n '10,20p' file` | Prints only lines 10 to 20. |
| `sed -n '/START/,/END/p'` | Prints the lines between two patterns. |
| `sed '5d'` | Deletes line 5. |
| `sed '/^#/d'` | Deletes comment lines. |
| `sed '/^$/d'` | Deletes empty lines. |
| `sed '/\[mysqld\]/a key=val'` | Appends a line after a matching line. |
| `sed '3i\text'` | Inserts a line before line 3. |
| `sed '/^PermitRootLogin/c\PermitRootLogin no'` | Replaces a whole matching line. |

### awk — column processing

| Command | What it does |
|---------|--------------|
| `awk '{print $1}' file` | Prints the first whitespace-separated column. |
| `awk '{print $NF}'` | Prints the last column. |
| `awk -F: '{print $1,$7}' /etc/passwd` | Prints columns using a custom separator. |
| `awk -F: '$3>=1000{print $1}' /etc/passwd` | Prints lines where a column meets a condition. |
| `awk '$9 ~ /^5/' access.log` | Prints lines where a column matches a regex (5xx). |
| `awk 'NR==5'` / `awk 'NR>1'` | Prints line 5 / skips the header line. |
| `awk '/ERROR/{c++} END{print c}'` | Counts lines matching a pattern. |
| `awk '{s+=$10} END{print s}'` | Sums a column. |
| `awk '{s+=$NF;n++} END{print s/n}'` | Calculates the average of a column. |
| `awk '{a[$1]++} END{for(k in a) print a[k],k}'` | Groups and counts by a column. |
| `awk '!seen[$0]++'` | Removes duplicates while keeping original order. |
| `awk 'length($0)>200'` | Prints lines longer than 200 characters. |
| `awk -v t=90 '$5+0>t'` | Passes a shell value into awk. |
| `awk 'BEGIN{OFS=","}{print $1,$2}'` | Outputs columns as CSV. |

### Other filters

| Command | What it does |
|---------|--------------|
| `tee -a file` | Appends piped output to a file while showing it. |
| `paste -d, a b` | Joins two files side by side. |
| `paste -sd, list` | Joins all lines into one comma-separated line. |
| `join -t, a.csv b.csv` | Joins two sorted files on a common field. |
| `column -t` | Aligns output into a neat table. |
| `column -t -s, data.csv` | Pretty-prints a CSV file. |
| `fold -w 80` | Wraps long lines at 80 characters. |
| `rev` | Reverses each line's characters. |
| `split -l 100000 big.log part_` | Splits a file every N lines. |
| `split -b 100M big.tar chunk_` | Splits a file into fixed-size chunks. |
| `seq 1 5` | Prints a sequence of numbers. |
| `shuf -n 1 hosts.txt` | Picks a random line. |
| `expand` / `unexpand` | Converts tabs to spaces / spaces to tabs. |

### Log analysis one-liners

| Command | What it does |
|---------|--------------|
| `awk '{print $1}' access.log \| sort \| uniq -c \| sort -rn \| head` | Shows the top 10 client IPs. |
| `awk '{print $9}' access.log \| sort \| uniq -c \| sort -rn` | Shows HTTP status code counts. |
| `awk '$9~/^5/{print $7}' access.log \| sort \| uniq -c \| sort -rn \| head` | Shows URLs returning the most 5xx errors. |
| `awk '{print substr($4,2,17)}' access.log \| uniq -c` | Shows requests per minute to spot spikes. |
| `grep -oE '[A-Za-z.]+Exception' app.log \| sort \| uniq -c \| sort -rn` | Ranks the most frequent exception types. |
| `grep "Failed password" /var/log/secure \| awk '{print $(NF-3)}' \| sort \| uniq -c \| sort -rn` | Finds IPs with the most failed SSH logins. |
| `grep -rl old /etc/app \| xargs sed -i.bak 's/old/new/g'` | Replaces a value across many files with backups. |

## 10. JSON & YAML

| Command | What it does |
|---------|--------------|
| `jq .` | Pretty-prints JSON. |
| `jq '.status'` | Extracts one field. |
| `jq -r '.items[].metadata.name'` | Extracts values from an array as plain text. |
| `jq -r '.items[] \| select(.status.phase!="Running") \| .metadata.name'` | Filters array items by a condition. |
| `jq -r '[.a,.b] \| @tsv'` | Outputs fields as tab-separated values. |
| `jq '.replicas = 3'` | Modifies a JSON field. |
| `jq -c '.[]'` | Prints each object compactly on one line. |
| `jq --arg v "$X" '.name=$v'` | Passes a shell variable into jq. |
| `yq '.spec.replicas' f.yaml` | Reads a value from YAML. |
| `yq -i '.image.tag="v2"' values.yaml` | Edits a YAML file in place. |

## 11. Archiving & Compression

### tar

| Command | What it does |
|---------|--------------|
| `tar -cvf out.tar dir` | Creates an uncompressed archive. |
| `tar -czvf out.tar.gz dir` | Creates a gzip-compressed archive (most common). |
| `tar -cjvf out.tar.bz2 dir` | Creates a bzip2-compressed archive. |
| `tar -cJvf out.tar.xz dir` | Creates an xz-compressed archive (smallest). |
| `tar --zstd -cvf out.tar.zst dir` | Creates a zstd-compressed archive (fast, modern). |
| `tar -czpf etc.tgz /etc` | Archives while preserving permissions. |
| `tar -czf app.tgz -C /opt app` | Changes directory first so paths stay clean. |
| `tar -czf out.tgz --exclude='*.tmp' dir` | Skips files matching a pattern. |
| `tar -tvf out.tar.gz` | Lists archive contents without extracting. |
| `tar -xvf out.tar` | Extracts an archive here (compression auto-detected). |
| `tar -xzf out.tgz -C /restore` | Extracts into a target directory. |
| `tar -xzf out.tgz path/to/file` | Extracts a single file. |
| `tar -xzf app.tgz --strip-components=1` | Extracts while removing the top-level folder. |
| `tar -xzf out.tgz --wildcards '*.conf'` | Extracts only files matching a pattern. |
| `tar -rvf out.tar newfile` | Appends a file to an uncompressed archive. |
| `tar -dvf out.tar` | Compares an archive with the filesystem. |
| `tar -I pigz -cf out.tgz dir` | Creates an archive using parallel gzip. |
| `tar --listed-incremental=snap -czf inc.tgz dir` | Creates incremental backups using a snapshot file. |

### gzip, bzip2, xz, zstd, pigz

| Command | What it does |
|---------|--------------|
| `gzip file` | Compresses a file to `.gz` and removes the original. |
| `gzip -k file` | Compresses and keeps the original. |
| `gzip -9` / `gzip -1` | Uses best compression / fastest compression. |
| `gzip -r dir` | Compresses every file in a directory. |
| `gzip -l file.gz` | Shows compression ratio. |
| `gzip -t file.gz` | Tests archive integrity. |
| `gunzip file.gz` / `gzip -d` | Decompresses a `.gz` file. |
| `gzip -dc file.gz > file` | Decompresses to stdout and keeps the `.gz`. |
| `zcat` / `zless` / `zgrep` | Reads / pages / searches `.gz` files without extracting. |
| `bzip2 -k` / `bunzip2` / `bzcat` | Compresses / decompresses / reads bzip2 files. |
| `xz -T0 -k file` | Compresses with xz using all CPU cores. |
| `xz -d` / `unxz` / `xzcat` | Decompresses / reads xz files. |
| `zstd -T0 -19 file` | Compresses with zstd at high level using all cores. |
| `zstd -d` / `unzstd` / `zstdcat` | Decompresses / reads zstd files. |
| `pigz -p 8 file` | Compresses with gzip using 8 threads. |
| `unpigz file.gz` | Decompresses with parallel gzip. |

### zip & others

| Command | What it does |
|---------|--------------|
| `zip -r site.zip dir` | Creates a zip archive of a directory. |
| `zip -r out.zip . -x "*.git*"` | Zips while excluding files (e.g., Lambda packages). |
| `zip -e secret.zip file` | Creates a password-protected zip. |
| `unzip file.zip` | Extracts a zip archive. |
| `unzip file.zip -d /opt/app` | Extracts into a target directory. |
| `unzip -l file.zip` | Lists zip contents. |
| `unzip -o file.zip` | Extracts and overwrites without prompting. |
| `unzip -t file.zip` | Tests the archive for errors. |
| `7z a out.7z dir` / `7z x out.7z` | Creates / extracts a 7-Zip archive. |
| `unrar x file.rar` | Extracts a RAR archive. |
| `rpm2cpio pkg.rpm \| cpio -idmv` | Unpacks an RPM without installing it. |

### Logrotate

| Command | What it does |
|---------|--------------|
| `logrotate -d /etc/logrotate.d/app` | Dry-runs a rotation config to show what would happen. |
| `logrotate -f /etc/logrotate.d/app` | Forces rotation immediately. |
| `logrotate -v /etc/logrotate.conf` | Runs rotation with verbose output. |
| `cat /var/lib/logrotate/logrotate.status` | Shows when each log was last rotated. |

## 12. File Transfer & Sync

| Command | What it does |
|---------|--------------|
| `rsync -avh src/ dest/` | Syncs a directory locally preserving attributes. |
| `rsync -avhz --progress src/ user@host:/dst/` | Syncs to a remote server over SSH with compression. |
| `rsync -avhn --delete src/ dest/` | Dry-runs a mirror to preview deletions. |
| `rsync -avh --delete src/ dest/` | ⚠️ Mirrors exactly, deleting extra files at destination. |
| `rsync -avh --exclude='.git' src/ dest/` | Syncs while skipping matching files. |
| `rsync -avhP bigfile host:/tmp/` | Transfers with a progress bar and resume support. |
| `rsync -e "ssh -p 2222 -i key" src/ host:/dst/` | Syncs using custom SSH options. |
| `rsync --bwlimit=5000` | Limits bandwidth used during sync. |
| `scp file user@host:/tmp/` | Copies a file to a remote server. |
| `scp user@host:/var/log/app.log .` | Copies a file from a remote server. |
| `scp -r dir host:/opt/` | Copies a directory to a remote server. |
| `scp -P 2222 -i key.pem file host:~` | Copies using a custom port and key. |
| `scp -3 host1:/f host2:/f` | Copies between two remote servers via your machine. |
| `sftp user@host` | Opens an interactive file transfer session. |
| `tar -czf - dir \| ssh host "cat > dir.tgz"` | Streams a compressed archive to a remote server. |
| `tar -cf - dir \| ssh host "tar -xf - -C /dst"` | Copies a directory to another server with no temp file. |
| `ssh host "tar -czf - /var/log/nginx" > logs.tgz` | Pulls remote logs as a compressed archive. |
| `mysqldump db \| gzip > db.sql.gz` | Dumps a database straight into a compressed file. |
| `gunzip < db.sql.gz \| mysql db` | Restores a database from a compressed dump. |
| `tar -czf - dir \| aws s3 cp - s3://bucket/x.tgz` | Uploads an archive to S3 without a local copy. |
| `tar -cf - dir \| pv \| gzip > out.tgz` | Shows a progress bar while archiving. |

## 13. Important Paths (Filesystem Hierarchy)

| Path | What it holds |
|------|---------------|
| `/etc` | System-wide configuration files. |
| `/etc/passwd`, `/etc/shadow`, `/etc/group` | User accounts, password hashes and groups. |
| `/etc/sudoers`, `/etc/sudoers.d/` | Sudo rules. |
| `/etc/fstab` | Filesystems mounted at boot. |
| `/etc/hosts`, `/etc/resolv.conf`, `/etc/nsswitch.conf` | Static hosts, DNS servers and lookup order. |
| `/etc/ssh/sshd_config` | SSH server configuration. |
| `/etc/systemd/system/` | Your custom unit files and overrides. |
| `/etc/sysctl.d/` | Kernel parameter settings. |
| `/etc/security/limits.conf` | User resource limits (open files, processes). |
| `/etc/yum.repos.d/`, `/etc/apt/sources.list.d/` | Package repository definitions. |
| `/var/log/` | System and application logs. |
| `/var/log/messages` / `/var/log/syslog` | General system log (RHEL / Ubuntu). |
| `/var/log/secure` / `/var/log/auth.log` | Login, SSH and sudo log (RHEL / Ubuntu). |
| `/var/log/cloud-init-output.log` | Output of cloud user-data scripts. |
| `/var/lib/` | Application state (docker, kubelet, mysql). |
| `/var/cache/`, `/var/spool/`, `/var/tmp/` | Caches, queues and persistent temp files. |
| `/proc/<PID>/` | Live info about a process (cmdline, fd, limits, status). |
| `/proc/meminfo`, `/proc/cpuinfo`, `/proc/loadavg` | Live memory, CPU and load data. |
| `/sys/class/net/`, `/sys/block/`, `/sys/fs/cgroup/` | Network devices, disks and cgroup limits. |
| `/dev/sda`, `/dev/nvme0n1`, `/dev/mapper/` | Disk devices and LVM/encrypted volumes. |
| `/dev/null`, `/dev/zero`, `/dev/urandom` | Discard sink, zero source, random source. |
| `/usr/bin`, `/usr/sbin` | Package-managed programs. |
| `/usr/local/bin`, `/opt` | Manually installed and vendor software. |
| `/boot` | Kernel, initramfs and bootloader files. |
| `/tmp`, `/run` | Temporary and runtime files cleared at reboot. |
| `/home/<user>`, `/root` | User home directories / root's home. |

| Command | What it does |
|---------|--------------|
| `cat /proc/PID/cmdline \| tr '\0' ' '` | Shows the full command a process was started with. |
| `tr '\0' '\n' < /proc/PID/environ` | Shows the environment variables of a process. |
| `ls -l /proc/PID/fd \| wc -l` | Counts open file descriptors of a process. |
| `cat /proc/PID/limits` | Shows the effective limits of a running process. |
| `ls -l /proc/PID/cwd` | Shows the working directory of a process. |
| `cat /proc/sys/fs/file-nr` | Shows system-wide open file handle usage. |
| `namei -l /path/to/file` | Shows permissions of every directory in a path. |

---

# Part 3 — Access & Security

## 14. Users & Groups

### Create, modify, delete users

| Command | What it does |
|---------|--------------|
| `useradd -m -s /bin/bash manoj` | Creates a user with a home directory and bash shell. |
| `useradd -m -s /bin/bash -c "Name" -G wheel,docker user` | Creates a user with a comment and extra groups. |
| `useradd -m -u 1500 -g devops user` | Creates a user with a specific UID and primary group. |
| `useradd -m -e 2026-12-31 contractor` | Creates a user whose account expires on a date. |
| `useradd -r -M -s /sbin/nologin appsvc` | Creates a system/service account with no login and no home. |
| `useradd -D` | Shows the defaults used for new users. |
| `adduser manoj` | Creates a user interactively (Debian/Ubuntu). |
| `usermod -aG docker manoj` | Adds a user to a group while keeping existing groups. |
| `usermod -G docker manoj` | ⚠️ Replaces all of a user's secondary groups. |
| `usermod -g devops manoj` | Changes a user's primary group. |
| `usermod -s /sbin/nologin user` | Disables interactive shell access. |
| `usermod -L user` / `usermod -U user` | Locks / unlocks the user's password. |
| `usermod -e 2026-12-31 user` | Sets an account expiry date. |
| `usermod -e 1 user` | Expires the account now, blocking all logins including SSH keys. |
| `usermod -l newname oldname` | Renames a login. |
| `usermod -d /new/home -m user` | Moves a user's home directory. |
| `usermod -u 2001 user` | Changes a user's UID. |
| `usermod -c "New comment" user` | Changes the user's full-name comment. |
| `userdel user` | Deletes a user but keeps the home directory. |
| `userdel -r user` | Deletes a user with home and mail spool. |
| `userdel -f user` | Deletes a user even if logged in. |
| `vipw` / `vipw -s` | Safely edits `/etc/passwd` / `/etc/shadow` with locking. |
| `vigr` | Safely edits `/etc/group`. |
| `pwck` / `grpck` | Checks user and group files for consistency. |

### Groups

| Command | What it does |
|---------|--------------|
| `groupadd devops` | Creates a group. |
| `groupadd -g 3000 sre` | Creates a group with a specific GID. |
| `groupadd -r appgrp` | Creates a system group. |
| `groupmod -n platform devops` | Renames a group. |
| `groupmod -g 3100 sre` | Changes a group's GID. |
| `groupdel oldgroup` | Deletes a group. |
| `gpasswd -a user group` | Adds a user to a group. |
| `gpasswd -d user group` | Removes a user from a group. |
| `gpasswd -M a,b,c group` | Sets the full member list of a group. |
| `gpasswd -A user group` | Makes a user the group administrator. |
| `newgrp docker` | Activates a newly added group in the current shell. |

### Query & audit

| Command | What it does |
|---------|--------------|
| `getent passwd user` | Looks up a user from all sources (local, LDAP, AD). |
| `getent group docker` | Shows a group and its members from all sources. |
| `getent group wheel sudo` | Shows who has sudo through admin groups. |
| `lastlog -b 90` | Lists users who have not logged in for 90+ days. |
| `awk -F: '$3>=1000 && $3<65534{print $1}' /etc/passwd` | Lists human (non-system) users. |
| `awk -F: '$3==0{print $1}' /etc/passwd` | Lists all UID 0 accounts (should be only root). |
| `awk -F: '$2==""{print $1}' /etc/shadow` | Finds accounts with empty passwords. |
| `awk -F: '$7!~/(nologin\|false)/{print $1,$7}' /etc/passwd` | Lists users with a login shell. |
| `realm list` | Shows Active Directory domain join status. |
| `realm join corp.example.com -U admin` | Joins the server to an AD domain. |
| `sss_cache -E` | Clears the SSSD cache so group changes appear. |

### Switching users

| Command | What it does |
|---------|--------------|
| `su - user` | Switches to another user with their full login environment. |
| `su -` | Becomes root using root's password. |
| `su -s /bin/bash -c "cmd" appsvc` | Runs a command as a no-login service user. |
| `exit` | Returns to the previous user. |

### Resource limits

| Command | What it does |
|---------|--------------|
| `ulimit -a` | Shows all limits for the current shell. |
| `ulimit -n` / `ulimit -Hn` | Shows the soft / hard limit on open files. |
| `ulimit -n 65535` | Raises the open-file limit for this session. |
| `ulimit -u` | Shows the max number of user processes. |
| `/etc/security/limits.d/90-app.conf` | Sets persistent per-user limits (`app hard nofile 65535`). |
| `LimitNOFILE=65535` (systemd unit) | Sets open-file limits for a systemd service. |

## 15. Passwords, Aging & Lockout

| Command | What it does |
|---------|--------------|
| `passwd` | Changes your own password. |
| `passwd user` | Sets another user's password (root). |
| `passwd -l user` / `passwd -u user` | Locks / unlocks a password (SSH keys still work). |
| `passwd -e user` | Forces a password change at next login. |
| `passwd -d user` | ⚠️ Removes the password entirely. |
| `passwd -S user` | Shows password status (set, locked, none). |
| `echo "user:Pass" \| chpasswd` | Sets a password non-interactively (automation). |
| `openssl passwd -6 'Pass'` | Generates a SHA-512 password hash. |
| `chage -l user` | Shows password aging information. |
| `chage -M 90 -m 1 -W 7 user` | Sets max 90 days, min 1 day, warn 7 days. |
| `chage -d 0 user` | Forces a password change at next login. |
| `chage -E 2026-12-31 user` | Sets an account expiry date. |
| `chage -E -1 user` | Removes account expiry. |
| `chage -I 14 user` | Locks the account 14 days after the password expires. |
| `faillock --user user` | Shows failed login attempts. |
| `faillock --user user --reset` | Unlocks an account locked after failed logins. |
| `pam_tally2 -u user -r` | Resets failed-login count on older systems. |
| `/etc/security/pwquality.conf` | Sets password complexity rules. |
| `/etc/login.defs` | Sets default password aging, UID ranges and umask. |

## 16. Permissions & Ownership

| Command | What it does |
|---------|--------------|
| `chmod 755 script.sh` | Owner rwx, group and others r-x. |
| `chmod 644 file` | Owner rw, everyone else read-only. |
| `chmod 640 file` | Owner rw, group read, others nothing. |
| `chmod 600 key` | Only the owner can read and write. |
| `chmod 400 key.pem` | Only the owner can read (AWS keys). |
| `chmod 700 ~/.ssh` | Only the owner can access the directory. |
| `chmod +x script.sh` | Makes a file executable for everyone. |
| `chmod u+x` / `g+w` / `o-rwx` | Adds or removes permissions for owner/group/others. |
| `chmod go= file` | Removes all permissions from group and others. |
| `chmod u=rwx,g=rx,o= dir` | Sets exact symbolic permissions. |
| `chmod -R g+rX dir` | Recursively adds read, and execute only on directories. |
| `chmod --reference=a b` | Copies permissions from another file. |
| `find dir -type d -exec chmod 755 {} +` | Sets all directories to 755. |
| `find dir -type f -exec chmod 644 {} +` | Sets all files to 644. |
| `chown user file` | Changes the file owner. |
| `chown user:group file` | Changes owner and group together. |
| `chown :group file` | Changes only the group. |
| `chown -R nginx:nginx /var/www` | Changes ownership recursively. |
| `chown -h user link` | Changes ownership of a symlink itself. |
| `chown -R 1001:1001 /data` | Sets numeric ownership (container volumes). |
| `chgrp -R devops dir` | Changes group ownership recursively. |
| `chmod 4755 file` / `chmod u+s` | Sets SUID so the file runs as its owner. |
| `chmod 2775 dir` / `chmod g+s` | Sets SGID so new files inherit the directory's group. |
| `chmod 1777 dir` / `chmod +t` | Sets the sticky bit so users can delete only their own files. |
| `umask` / `umask -S` | Shows default permission mask (numeric / symbolic). |
| `umask 027` | New files 640 and new dirs 750 for this session. |

## 17. ACLs, Attributes, SELinux & Capabilities

| Command | What it does |
|---------|--------------|
| `getfacl path` | Shows extended ACL entries. |
| `setfacl -m u:auditor:rx path` | Gives one extra user specific access. |
| `setfacl -m g:qa:rwx path` | Gives one extra group specific access. |
| `setfacl -R -m u:jenkins:rwX dir` | Applies an ACL recursively. |
| `setfacl -d -m g:devops:rwx dir` | Sets a default ACL inherited by new files. |
| `setfacl -x u:auditor path` | Removes one ACL entry. |
| `setfacl -b path` | Removes all ACL entries. |
| `chattr +i file` | Makes a file immutable, even for root. |
| `chattr -i file` | Removes the immutable flag. |
| `chattr +a file` | Makes a file append-only (tamper-resistant logs). |
| `lsattr file` | Shows special file attributes. |
| `getenforce` / `sestatus` | Shows SELinux mode / detailed status. |
| `setenforce 0` | Sets SELinux to permissive temporarily (debug only). |
| `ls -Z` / `ps -eZ` | Shows SELinux labels of files / processes. |
| `restorecon -Rv /path` | Resets files to their default SELinux context (common fix). |
| `chcon -t httpd_sys_content_t -R /path` | Changes SELinux context temporarily. |
| `semanage fcontext -a -t TYPE "/path(/.*)?"` | Adds a permanent SELinux file context rule. |
| `semanage port -a -t http_port_t -p tcp 8081` | Allows a service to use a new port under SELinux. |
| `getsebool -a \| grep httpd` | Lists SELinux booleans for a service. |
| `setsebool -P httpd_can_network_connect on` | Permanently enables an SELinux boolean. |
| `ausearch -m avc -ts recent` | Shows recent SELinux denials. |
| `sealert -a /var/log/audit/audit.log` | Explains SELinux denials in plain language. |
| `aa-status` | Shows loaded AppArmor profiles (Ubuntu). |
| `aa-complain` / `aa-enforce /usr/sbin/nginx` | Sets an AppArmor profile to log-only / enforce. |
| `getcap file` / `getcap -r /` | Shows Linux capabilities on a file / all files. |
| `setcap 'cap_net_bind_service=+ep' bin` | Lets a program bind to ports below 1024 without root. |
| `setcap -r bin` | Removes capabilities from a file. |

## 18. Sudo & Privilege Management

### Using sudo

| Command | What it does |
|---------|--------------|
| `sudo cmd` | Runs one command as root and logs it. |
| `sudo -u postgres psql` | Runs a command as another user. |
| `sudo -i` | Opens a root login shell with root's environment. |
| `sudo -s` | Opens a root shell keeping your environment. |
| `sudo -u appsvc -i` | Opens a shell as a service account. |
| `sudo -l` | Lists what you are allowed to run with sudo. |
| `sudo -l -U user` | Lists another user's sudo rights. |
| `sudo -v` | Refreshes your cached sudo credentials. |
| `sudo -k` | Forgets cached credentials so the next sudo asks again. |
| `sudo -n true` | Tests whether sudo works without a password (scripts). |
| `sudo -E cmd` | Runs sudo preserving your environment (if allowed). |
| `sudo -H cmd` | Sets HOME to the target user's home. |
| `sudo -b cmd` | Runs the command in the background. |
| `sudoedit file` / `sudo -e` | Edits a root file safely as yourself. |
| `runuser -u appsvc -- cmd` | Root runs a command as another user without a prompt. |

### Granting & editing rules

| Command | What it does |
|---------|--------------|
| `usermod -aG wheel user` | Grants full sudo on RHEL-family systems. |
| `usermod -aG sudo user` | Grants full sudo on Ubuntu/Debian. |
| `visudo` | Edits `/etc/sudoers` with syntax checking. |
| `visudo -f /etc/sudoers.d/team` | Edits a drop-in sudo file safely (best practice). |
| `visudo -c` | Checks all sudoers files for syntax errors. |
| `install -m 0440 -o root -g root f /etc/sudoers.d/f` | Installs a drop-in with correct permissions. |
| `user ALL=(root) NOPASSWD: /usr/bin/systemctl restart app` | Allows one exact command with no password. |
| `%devops ALL=(ALL) ALL` | Grants full sudo to a group. |
| `%dba ALL=(postgres) NOPASSWD: ALL` | Allows acting only as a specific user. |
| `Cmnd_Alias SVC = /usr/bin/systemctl restart nginx` | Groups commands under one name. |
| `User_Alias` / `Host_Alias` / `Runas_Alias` | Groups users / hosts / target users under one name. |
| `Defaults use_pty` | Runs sudo commands inside a pseudo-terminal. |
| `Defaults logfile="/var/log/sudo.log"` | Writes sudo events to a dedicated log. |
| `Defaults log_input, log_output` | Records full sudo sessions for audit. |
| `Defaults timestamp_timeout=5` | Asks for the password again after 5 minutes. |
| `Defaults secure_path=...` | Sets the trusted PATH used by sudo. |
| `echo "rm -f /etc/sudoers.d/x" \| at now + 4 hours` | Auto-revokes temporary sudo access. |

### Auditing sudo

| Command | What it does |
|---------|--------------|
| `grep sudo /var/log/secure` | Shows sudo activity on RHEL. |
| `grep sudo /var/log/auth.log` | Shows sudo activity on Ubuntu. |
| `journalctl _COMM=sudo --since today` | Shows today's sudo activity from the journal. |
| `sudoreplay -l` / `sudoreplay ID` | Lists / replays recorded sudo sessions. |
| `auditctl -w /etc/sudoers -p wa -k sudoers_change` | Audits any change to the sudoers file. |
| `ausearch -k sudoers_change` | Shows recorded sudoers changes. |
| `passwd -l root` | Locks the root password. |

## 19. SSH & Remote Access

### Connecting

| Command | What it does |
|---------|--------------|
| `ssh user@host` | Connects to a remote server. |
| `ssh -i key.pem user@host` | Connects using a specific private key. |
| `ssh -p 2222 user@host` | Connects on a custom port. |
| `ssh host "uptime; df -h"` | Runs remote commands and returns. |
| `ssh -t host "sudo systemctl restart nginx"` | Forces a terminal so remote sudo prompts work. |
| `ssh -J bastion user@private-host` | Connects through a jump host. |
| `ssh -v` / `ssh -vvv` | Shows debug output to troubleshoot connections. |
| `ssh -o ConnectTimeout=5 -o BatchMode=yes host` | Fails fast without prompts (scripts). |
| `ssh -o StrictHostKeyChecking=accept-new host` | Accepts new host keys but rejects changed ones. |
| `ssh -o IdentitiesOnly=yes -i key host` | Offers only the specified key. |
| `ssh -A host` | Forwards your SSH agent (use with care). |
| `ssh -C host` | Compresses traffic on slow links. |
| `ssh -X host` | Forwards GUI (X11) applications. |
| `Enter ~ .` | Kills a frozen SSH session. |

### Keys & agent

| Command | What it does |
|---------|--------------|
| `ssh-keygen -t ed25519 -C "user@laptop"` | Generates a modern ed25519 key pair. |
| `ssh-keygen -t rsa -b 4096` | Generates an RSA key for older systems. |
| `ssh-keygen -f ~/.ssh/prod_key` | Generates a key with a custom file name. |
| `ssh-keygen -p -f key` | Adds or changes a key's passphrase. |
| `ssh-keygen -y -f key > key.pub` | Recreates the public key from a private key. |
| `ssh-keygen -lf key.pub` | Shows a key's fingerprint. |
| `ssh-keygen -lf ~/.ssh/authorized_keys` | Shows fingerprints of all authorized keys. |
| `ssh-keygen -R host` | Removes a host from `known_hosts`. |
| `ssh-keygen -A` | Generates missing server host keys. |
| `ssh-copy-id -i key.pub user@host` | Installs your public key on a server. |
| `eval "$(ssh-agent -s)"` | Starts the SSH agent. |
| `ssh-add key` | Loads a key into the agent (passphrase asked once). |
| `ssh-add -t 8h key` | Loads a key that auto-expires after 8 hours. |
| `ssh-add -l` | Lists keys loaded in the agent. |
| `ssh-add -D` | Removes all keys from the agent. |
| `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys ~/.ssh/id_*` | Sets the permissions SSH requires. |
| `restorecon -Rv ~/.ssh` | Fixes SELinux labels on SSH files (RHEL). |

### Tunnels & file copy

| Command | What it does |
|---------|--------------|
| `ssh -N -L 5432:db:5432 bastion` | Makes a remote database reachable on your local port. |
| `ssh -N -R 9000:localhost:3000 host` | Exposes your local port on the remote server. |
| `ssh -N -D 1080 bastion` | Creates a SOCKS proxy through the server. |
| `ssh -f -N -L ...` | Runs a tunnel in the background. |
| `scp -J bastion file host:/tmp/` | Copies a file through a jump host. |
| `ssh-keyscan -t ed25519 host >> ~/.ssh/known_hosts` | Pre-loads a host key (CI pipelines). |

### Client config (`~/.ssh/config`)

| Directive | What it does |
|-----------|--------------|
| `Host alias` | Defines a short name and its settings. |
| `HostName` / `User` / `Port` | Sets the real address, username and port. |
| `IdentityFile ~/.ssh/key` | Sets which key to use. |
| `ProxyJump bastion` | Always connects through a bastion. |
| `LocalForward 5432 localhost:5432` | Opens a tunnel automatically. |
| `ServerAliveInterval 60` | Sends keepalives so idle sessions don't drop. |
| `ControlMaster auto` + `ControlPersist 10m` | Reuses one connection for faster repeat logins. |

### SSH certificates, fleet & cloud access

| Command | What it does |
|---------|--------------|
| `ssh-keygen -s ca_key -I id -n user -V +8h key.pub` | Signs a short-lived SSH user certificate. |
| `ssh-keygen -Lf key-cert.pub` | Shows certificate details. |
| `for h in web0{1..5}; do ssh $h uptime; done` | Runs a command across several servers. |
| `pdsh -w web[01-10] "cmd"` / `parallel-ssh -h hosts "cmd"` | Runs a command on many servers in parallel. |
| `ansible all -m ping` | Checks connectivity to all inventory hosts. |
| `ansible web -m shell -a "uptime"` | Runs an ad-hoc command across a group. |
| `aws ssm start-session --target i-0abc` | Opens a shell on EC2 without SSH or open ports. |
| `gcloud compute ssh vm --tunnel-through-iap` | Connects to a GCP VM through IAP. |

## 20. SSH Server Hardening & Troubleshooting

| Command / Setting | What it does |
|-------------------|--------------|
| `PermitRootLogin no` | Blocks direct root login over SSH. |
| `PasswordAuthentication no` | Allows key-based login only. |
| `AllowGroups ssh-users sre` | Allows only listed groups to log in. |
| `MaxAuthTries 3` | Limits authentication attempts per connection. |
| `ClientAliveInterval 300` | Disconnects idle sessions. |
| `AllowTcpForwarding no` / `X11Forwarding no` | Disables tunnels / GUI forwarding. |
| `LogLevel VERBOSE` | Logs the key fingerprint used for each login. |
| `UseDNS no` | Speeds up logins by skipping reverse DNS. |
| `sshd -t` | Tests the SSH server config for errors. |
| `sshd -T` | Prints the full effective SSH server config. |
| `systemctl reload sshd` | Applies config without dropping existing sessions. |
| `nc -zv host 22` | Checks whether the SSH port is reachable. |
| `ss -tlnp \| grep ssh` | Checks whether sshd is listening. |
| `tail -f /var/log/secure` / `auth.log` | Watches SSH login attempts live. |
| `journalctl -u sshd -f` | Follows SSH server logs. |
| `fail2ban-client status sshd` | Shows IPs banned for SSH brute force. |
| `fail2ban-client set sshd unbanip IP` | Unbans an IP. |
| `ssh -o HostKeyAlgorithms=+ssh-rsa host` | Connects to very old servers using legacy key types. |

## 21. Offboarding (Access Removal)

| Command | What it does |
|---------|--------------|
| `last user` / `who \| grep user` | Checks recent and current logins for the user. |
| `usermod -L user` | Locks the password. |
| `usermod -e 1 user` | Expires the account so SSH keys stop working too. |
| `usermod -s /sbin/nologin user` | Removes shell access. |
| `pkill -KILL -u user` | Kills all of the user's processes. |
| `loginctl terminate-user user` | Ends all of the user's sessions. |
| `mv /home/user/.ssh/authorized_keys /root/backup/` | Removes the user's SSH keys and keeps evidence. |
| `grep -rl "user@" /home/*/.ssh/authorized_keys` | Finds the user's key on other accounts. |
| `gpasswd -d user wheel` | Removes sudo-group membership. |
| `grep -rn user /etc/sudoers /etc/sudoers.d/` | Finds sudo rules mentioning the user. |
| `rm -f /etc/sudoers.d/user && visudo -c` | Removes their sudo drop-in and validates sudoers. |
| `for g in $(id -nG user); do gpasswd -d user $g; done` | Removes the user from every group. |
| `crontab -l -u user` / `crontab -r -u user` | Reviews / removes the user's cron jobs. |
| `atq` | Checks for scheduled one-off jobs. |
| `grep -rl "User=user" /etc/systemd/system/` | Finds services that run as the user. |
| `tar -czpf /archive/user_home.tgz /home/user` | Archives the home directory for retention. |
| `userdel -r user` | Deletes the account after the retention period. |
| `find / -xdev -user user -o -nouser` | Finds files left behind by the user. |
| `sudo -l -U user; chage -l user` | Verifies no sudo rights and the account is expired. |

---

# Part 4 — Software & Services

## 22. Package Management — RHEL (dnf / yum / rpm)

### dnf / yum

| Command | What it does |
|---------|--------------|
| `dnf install -y nginx` | Installs a package without prompting. |
| `dnf install nginx-1.24.0` | Installs a specific version. |
| `dnf install ./app.rpm` | Installs a local RPM and resolves its dependencies. |
| `dnf reinstall nginx` | Reinstalls a package to repair its files. |
| `dnf remove nginx` | Removes a package. |
| `dnf autoremove` | Removes dependencies no longer needed. |
| `dnf downgrade nginx` | Installs the previous version of a package. |
| `dnf check-update` | Lists available updates (exit code 100 means updates exist). |
| `dnf update` | Updates all packages. |
| `dnf update nginx` | Updates a single package. |
| `dnf update --security` | Installs only security updates. |
| `dnf update --exclude=kernel*` | Updates everything except matching packages. |
| `dnf upgrade-minimal --security` | Applies the smallest update that fixes security issues. |
| `dnf update --downloadonly --downloaddir=/tmp/p` | Downloads updates without installing. |
| `dnf search nginx` | Searches package names and summaries. |
| `dnf info nginx` | Shows package details. |
| `dnf list installed` | Lists installed packages. |
| `dnf list --showduplicates nginx` | Lists every available version. |
| `dnf provides /usr/bin/netstat` | Finds which package provides a file. |
| `dnf repoquery -l nginx` | Lists files in a package. |
| `dnf repoquery --requires nginx` | Lists a package's dependencies. |
| `dnf repoquery --whatrequires openssl` | Lists packages that depend on this one. |
| `dnf repolist` / `dnf repolist all` | Lists enabled / all repositories. |
| `dnf config-manager --add-repo URL` | Adds a repository. |
| `dnf config-manager --set-enabled crb` | Enables a repository. |
| `dnf install epel-release` | Enables the EPEL extra packages repository. |
| `dnf --enablerepo=epel install htop` | Uses a repository for one command only. |
| `dnf group install "Development Tools"` | Installs a package group. |
| `dnf module enable nodejs:20` | Selects a module stream version. |
| `dnf clean all` | Clears cached metadata and packages. |
| `dnf makecache` | Rebuilds the metadata cache. |
| `dnf history` | Lists all past package transactions. |
| `dnf history info 42` | Shows what transaction 42 changed. |
| `dnf history undo 42` | Rolls back a specific transaction. |
| `dnf history rollback 40` | Returns the system to the state after transaction 40. |
| `dnf versionlock add pkg` | Prevents a package from being upgraded. |
| `dnf versionlock list` / `delete pkg` | Lists / removes version locks. |
| `dnf updateinfo summary` | Summarizes pending security advisories. |
| `dnf updateinfo list --security` | Lists pending security advisories. |
| `dnf update --advisory=RHSA-XXXX` | Applies one specific advisory. |
| `dnf update --cve CVE-XXXX` | Applies the fix for a specific CVE. |
| `dnf download --resolve --destdir=/repo pkg` | Downloads a package and its dependencies for offline use. |
| `createrepo_c /repo` | Builds a local repository from RPM files. |
| `dnf reposync --repoid=baseos -p /mirror` | Mirrors a whole repository locally. |
| `subscription-manager repos --list-enabled` | Lists enabled RHEL subscription repos. |

### rpm

| Command | What it does |
|---------|--------------|
| `rpm -qa` | Lists all installed packages. |
| `rpm -qa --last \| head` | Lists the most recently installed or updated packages. |
| `rpm -q nginx` | Shows whether a package is installed and its version. |
| `rpm -qi nginx` | Shows package details. |
| `rpm -ql nginx` | Lists files installed by a package. |
| `rpm -qc nginx` | Lists a package's config files. |
| `rpm -qf /usr/sbin/nginx` | Finds which package owns a file. |
| `rpm -qR nginx` | Lists a package's requirements. |
| `rpm -q --changelog openssl` | Shows the changelog (confirm CVE fixes). |
| `rpm -qpi app.rpm` / `rpm -qpl app.rpm` | Shows info / files of an uninstalled RPM file. |
| `rpm -V nginx` | Verifies installed files against the package (detects drift). |
| `rpm -Va` | Verifies every installed package (security audit). |
| `rpm -ivh app.rpm` | Installs an RPM without resolving dependencies. |
| `rpm -Uvh app.rpm` | Upgrades or installs an RPM. |
| `rpm -e app` | Removes a package. |
| `rpm --import key.gpg` | Imports a repository signing key. |
| `rpm -K app.rpm` | Checks an RPM's signature. |
| `rpm --rebuilddb` | Repairs a corrupted RPM database. |

## 23. Package Management — Ubuntu (apt / dpkg)

### apt

| Command | What it does |
|---------|--------------|
| `apt update` | Refreshes the package index (always run first). |
| `apt upgrade` | Upgrades installed packages without removing any. |
| `apt full-upgrade` | Upgrades and allows removals to resolve changes. |
| `apt install -y nginx` | Installs a package without prompting. |
| `apt install nginx=1.24.0-1` | Installs a specific version. |
| `apt install ./app.deb` | Installs a local `.deb` with dependencies. |
| `apt install --only-upgrade openssl` | Upgrades a package only if already installed. |
| `apt install --no-install-recommends pkg` | Installs without extra recommended packages. |
| `apt reinstall nginx` | Reinstalls a package. |
| `apt remove nginx` | Removes a package but keeps its config. |
| `apt purge nginx` | Removes a package and its config files. |
| `apt autoremove --purge` | Removes unused dependencies and their config. |
| `apt clean` / `apt autoclean` | Deletes all / obsolete cached package files. |
| `apt search nginx` | Searches for packages. |
| `apt show nginx` | Shows package details. |
| `apt list --installed` | Lists installed packages. |
| `apt list --upgradable` | Lists packages with pending updates. |
| `apt policy nginx` | Shows installed vs candidate version and source repo. |
| `apt-cache madison nginx` | Lists all available versions. |
| `apt-cache depends` / `rdepends pkg` | Shows dependencies / reverse dependencies. |
| `apt-file search /usr/bin/netstat` | Finds which package provides a file. |
| `apt-mark hold pkg` / `unhold pkg` | Pins / unpins a package version. |
| `apt-mark showhold` | Lists held packages. |
| `apt -f install` | Fixes broken dependencies. |
| `apt-get install -y -qq pkg` | Quiet non-interactive install for scripts. |
| `DEBIAN_FRONTEND=noninteractive` | Prevents install prompts in automation. |
| `add-apt-repository ppa:name/ppa` | Adds a PPA repository. |
| `gpg --dearmor -o /etc/apt/keyrings/x.gpg` | Stores a repo key for `signed-by` use. |
| `apt-get install --download-only pkg` | Downloads packages without installing. |

### dpkg

| Command | What it does |
|---------|--------------|
| `dpkg -l` | Lists installed packages with status codes. |
| `dpkg -s nginx` | Shows a package's status and details. |
| `dpkg -L nginx` | Lists files installed by a package. |
| `dpkg -S /usr/sbin/nginx` | Finds which package owns a file. |
| `dpkg -c app.deb` / `dpkg -I app.deb` | Shows contents / info of a `.deb` file. |
| `dpkg -i app.deb` | Installs a `.deb` without resolving dependencies. |
| `dpkg -r app` / `dpkg -P app` | Removes / purges a package. |
| `dpkg --configure -a` | Finishes interrupted installs (common fix). |
| `dpkg-reconfigure tzdata` | Re-runs a package's configuration. |
| `grep " install " /var/log/dpkg.log` | Shows package install history. |
| `grep -E "Start-Date\|Commandline" /var/log/apt/history.log` | Shows what apt changed and when. |

## 24. Other Package Tools, Patching & Kernel

| Command | What it does |
|---------|--------------|
| `zypper refresh` / `install` / `update` / `patch` | Manages packages on SUSE. |
| `apk update` / `apk add --no-cache curl` / `apk del` | Manages packages on Alpine (containers). |
| `pacman -Syu` / `pacman -S pkg` | Upgrades / installs packages on Arch. |
| `snap install kubectl --classic` / `snap list` | Installs / lists snap packages. |
| `python3 -m venv .venv` | Creates an isolated Python environment. |
| `pip install -r requirements.txt` | Installs Python dependencies. |
| `pip freeze > requirements.txt` | Saves exact Python package versions. |
| `pipx install ansible-core` | Installs a Python CLI tool in isolation. |
| `npm ci` | Installs exact Node dependencies from the lockfile. |
| `go install pkg@latest` | Installs a Go tool. |
| `install -m 0755 kubectl /usr/local/bin/` | Installs a single downloaded binary. |
| `./configure && make -j$(nproc) && sudo make install` | Builds software from source. |
| `unattended-upgrade --dry-run -d` | Previews automatic security upgrades (Ubuntu). |
| `unattended-upgrade` | Applies security upgrades now (Ubuntu). |
| `systemctl enable --now dnf-automatic.timer` | Enables automatic updates (RHEL). |
| `pro fix CVE-XXXX` | Fixes a CVE with Ubuntu Pro. |
| `rpm -q kernel` / `dpkg -l \| grep linux-image` | Lists installed kernels. |
| `grubby --default-kernel` | Shows which kernel boots by default. |
| `grubby --set-default /boot/vmlinuz-X` | Boots an older kernel next time (rollback). |
| `dnf remove --oldinstallonly --setopt installonly_limit=2 kernel` | Removes old kernels, keeping two. |
| `needs-restarting -r` | Reports whether a reboot is required (RHEL). |
| `needs-restarting -s` | Lists services needing a restart (RHEL). |
| `cat /var/run/reboot-required` | Shows whether a reboot is required (Ubuntu). |
| `needrestart -r l` | Lists services needing a restart (Ubuntu). |

## 25. Services (systemd)

| Command | What it does |
|---------|--------------|
| `systemctl start svc` | Starts a service now. |
| `systemctl stop svc` | Stops a service now. |
| `systemctl restart svc` | Stops and starts a service (brief downtime). |
| `systemctl reload svc` | Reloads config without stopping the service. |
| `systemctl reload-or-restart svc` | Reloads if supported, otherwise restarts. |
| `systemctl enable svc` | Starts the service automatically at boot. |
| `systemctl enable --now svc` | Enables at boot and starts it now. |
| `systemctl disable --now svc` | Disables at boot and stops it now. |
| `systemctl mask svc` / `unmask` | Blocks / unblocks a service from ever starting. |
| `systemctl status svc` | Shows state, PID, memory and recent logs. |
| `systemctl status svc -l --no-pager` | Shows full status lines without a pager. |
| `systemctl is-active svc` | Prints active/inactive (use in scripts). |
| `systemctl is-enabled svc` | Prints whether the service starts at boot. |
| `systemctl --failed` | Lists all failed units (first incident check). |
| `systemctl list-units --type=service --state=running` | Lists running services. |
| `systemctl list-unit-files --state=enabled` | Lists services enabled at boot. |
| `systemctl cat svc` | Shows the unit file plus any overrides. |
| `systemctl show svc -p MainPID,Restart` | Prints specific unit properties. |
| `systemctl list-dependencies svc` | Shows what a unit depends on. |
| `systemctl daemon-reload` | Reloads unit files after you edit them. |
| `systemctl reset-failed` | Clears the failed state of units. |
| `systemctl kill -s SIGHUP svc` | Sends a signal to a service's processes. |
| `systemctl edit svc` | Creates an override file without touching the original. |
| `systemctl edit --full svc` | Edits a full copy of the unit file. |
| `systemctl --user status svc` | Manages a user-level service. |
| `systemd-analyze verify unit.service` | Checks a unit file for errors. |
| `systemd-delta` | Lists all overridden unit files. |

## 26. Scheduling & Boot

| Command | What it does |
|---------|--------------|
| `crontab -e` | Edits your cron jobs. |
| `crontab -l` | Lists your cron jobs. |
| `crontab -r` | ⚠️ Removes all your cron jobs. |
| `crontab -u user -l` | Lists another user's cron jobs. |
| `*/5 * * * * cmd` | Cron schedule that runs every 5 minutes. |
| `0 2 * * * cmd` | Cron schedule that runs daily at 02:00. |
| `@reboot cmd` | Cron schedule that runs at boot. |
| `systemctl list-timers --all` | Lists systemd timers and next run times. |
| `systemctl enable --now backup.timer` | Enables a systemd timer. |
| `systemd-analyze calendar "Mon..Fri 09:00"` | Validates a timer schedule expression. |
| `echo "cmd" \| at 02:00` | Schedules a one-time job. |
| `atq` / `atrm N` | Lists / removes one-time jobs. |
| `systemctl get-default` | Shows the default boot target. |
| `systemctl set-default multi-user.target` | Boots to text mode by default. |
| `systemctl isolate rescue.target` | Switches to single-user rescue mode. |
| `systemd-analyze` | Shows total boot time. |
| `systemd-analyze blame` | Lists units by how long they took to start. |
| `systemd-analyze critical-chain` | Shows the slowest boot dependency chain. |
| `systemctl reboot` / `poweroff` | Reboots / shuts down the system. |
| `shutdown -r +10 "msg"` | Schedules a reboot in 10 minutes with a warning. |
| `shutdown -c` | Cancels a scheduled shutdown. |

## 27. Logs

| Command | What it does |
|---------|--------------|
| `journalctl -u svc` | Shows all logs for a service. |
| `journalctl -u svc -f` | Follows a service's logs live. |
| `journalctl -u svc -n 100 --no-pager` | Shows the last 100 lines without a pager. |
| `journalctl -u svc --since "1 hour ago"` | Shows logs from the last hour. |
| `journalctl --since "10:00" --until "10:30"` | Shows logs within a time window. |
| `journalctl -p err -b` | Shows errors since the current boot. |
| `journalctl -b -1` | Shows logs from the previous boot (after a crash). |
| `journalctl -k` | Shows kernel messages. |
| `journalctl _PID=1234` | Shows logs from one process. |
| `journalctl -u svc -o json-pretty` | Shows logs as structured JSON. |
| `journalctl -u svc -o cat` | Shows only message text. |
| `journalctl -r` | Shows newest entries first. |
| `journalctl --list-boots` | Lists recorded boots. |
| `journalctl --disk-usage` | Shows how much disk the journal uses. |
| `journalctl --vacuum-size=1G` | Shrinks the journal to 1 GB. |
| `journalctl --vacuum-time=7d` | Deletes journal entries older than 7 days. |
| `dmesg -T` | Shows kernel messages with readable timestamps. |
| `dmesg -T \| tail -20` | Shows the latest kernel errors (OOM, disk). |
| `tail -F /var/log/app.log` | Follows an app log across rotations. |
| `zgrep -h ERROR app.log*` | Searches current and rotated compressed logs. |
| `less +F file` | Views a log in follow mode with search. |

---

# Part 5 — Processes & Performance

## 28. Viewing Processes

| Command | What it does |
|---------|--------------|
| `ps aux` | Lists all processes with CPU and memory usage. |
| `ps -ef` | Lists all processes in full format with parent PIDs. |
| `ps aux --sort=-%cpu \| head` | Shows the top CPU-consuming processes. |
| `ps aux --sort=-%mem \| head` | Shows the top memory-consuming processes. |
| `ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu` | Shows custom columns sorted by CPU. |
| `ps -p 1234 -o pid,etime,lstart,cmd` | Shows how long a process has been running. |
| `ps -u nginx` | Lists processes owned by a user. |
| `ps -fC java` | Lists processes by command name. |
| `ps -T -p 1234` | Lists threads of a process. |
| `ps -eLf \| wc -l` | Counts total threads on the system. |
| `ps axjf` / `pstree -p` | Shows processes as a tree. |
| `pgrep -a nginx` | Finds PIDs and full commands by name. |
| `pgrep -u app -f "java.*order"` | Finds processes by full command line and user. |
| `pidof nginx` | Prints the PIDs of a program. |
| `top` | Shows live process and system usage. |
| `top -o %MEM` | Sorts `top` by memory. |
| `top -p 1234` | Monitors specific processes. |
| `top -u nginx` | Shows only one user's processes. |
| `top -b -n 1 \| head -20` | Takes a one-time snapshot (for tickets/scripts). |
| `top -H -p 1234` | Shows threads of one process (find hot threads). |
| `htop` | Interactive, colored process viewer with tree view. |
| `atop` / `btop` / `glances` | Alternative monitors (atop keeps history). |

## 29. Signals, Jobs & Priority

| Command | What it does |
|---------|--------------|
| `kill PID` | Asks a process to stop gracefully (SIGTERM). |
| `kill -9 PID` | ⚠️ Force-kills a process immediately (last resort). |
| `kill -HUP PID` | Tells a process to reload its config. |
| `kill -3 PID` | Sends SIGQUIT (Java prints a thread dump). |
| `kill -STOP PID` / `kill -CONT PID` | Pauses / resumes a process. |
| `kill -0 PID` | Checks whether a process exists. |
| `kill -l` | Lists all signal names. |
| `pkill nginx` | Kills processes by name. |
| `pkill -f "python app.py"` | Kills processes by full command line. |
| `pkill -u user` | Kills all processes of a user. |
| `killall -9 java` | Force-kills all processes with an exact name. |
| `timeout 30s cmd` | Kills a command if it runs longer than 30 seconds. |
| `cmd &` | Starts a command in the background. |
| `jobs -l` | Lists background jobs with PIDs. |
| `fg %1` / `bg %1` | Brings a job to foreground / resumes it in background. |
| `disown -h %1` | Detaches a job so it survives logout. |
| `nohup cmd > out.log 2>&1 &` | Runs a command that keeps running after logout. |
| `setsid cmd` | Runs a command fully detached in a new session. |
| `tmux new -s name` | Starts a persistent terminal session. |
| `Ctrl+b d` / `tmux attach -t name` | Detaches from / reattaches to a tmux session. |
| `tmux ls` | Lists tmux sessions. |
| `screen -S name` / `screen -r name` | Starts / resumes a screen session. |
| `nice -n 10 cmd` | Starts a command with lower CPU priority. |
| `renice -n -5 -p PID` | Changes priority of a running process. |
| `renice -n 15 -u user` | Lowers priority of all a user's processes. |
| `ionice -c3 -p PID` | Gives a process idle-only disk priority. |
| `nice -n 19 ionice -c3 rsync ...` | Runs a background copy with minimal impact. |
| `taskset -cp 0,1 PID` | Pins a process to specific CPU cores. |

## 30. Deep Process Inspection

| Command | What it does |
|---------|--------------|
| `lsof -p PID` | Lists every file and socket a process has open. |
| `lsof -i :8080` | Shows which process uses port 8080. |
| `lsof -i TCP -s TCP:LISTEN` | Lists listening TCP sockets. |
| `lsof -i @10.0.1.5` | Lists connections to a host. |
| `lsof -u nginx` | Lists files opened by a user. |
| `lsof /var/log/app.log` | Shows who has a file open. |
| `lsof +D /data` | Lists everything open under a directory (unmount busy). |
| `lsof +L1` | Finds deleted files still held open (space not freed). |
| `lsof -nP -i` | Lists network connections quickly without name lookups. |
| `lsof -t -i :8080` | Prints only PIDs (for piping to kill). |
| `fuser -v 8080/tcp` | Shows which process uses a port. |
| `fuser -vm /data` | Shows processes using a mount. |
| `fuser -km /data` | ⚠️ Kills processes using a mount. |
| `strace -p PID` | Traces system calls of a running process. |
| `strace -f -p PID` | Traces a process and its threads/children. |
| `strace -e trace=file cmd` | Traces only file-related calls. |
| `strace -e trace=network -p PID` | Traces only network calls. |
| `strace -c -p PID` | Shows a summary count of system calls. |
| `strace -tt -T -o trace.txt cmd` | Traces with timestamps and call durations to a file. |
| `strace -f cmd 2>&1 \| grep ENOENT` | Finds missing files a program looks for. |
| `ltrace cmd` | Traces library calls. |
| `perf top -p PID` | Shows the hottest functions using CPU. |
| `gdb -p PID` | Attaches a debugger to a running process. |
| `jstack PID` / `jcmd PID Thread.print` | Prints a Java thread dump. |
| `py-spy top --pid PID` | Profiles a running Python process. |
| `pmap -x PID` | Shows a process's memory map. |

## 31. Performance & Memory

| Command | What it does |
|---------|--------------|
| `uptime` | Shows load averages to compare against `nproc`. |
| `vmstat 1 5` | Shows CPU, memory, swap and I/O every second, 5 times. |
| `vmstat -s` | Shows a memory statistics summary. |
| `mpstat -P ALL 1` | Shows usage for each CPU core. |
| `pidstat 1` | Shows CPU usage per process. |
| `pidstat -r 1` | Shows memory usage per process. |
| `pidstat -d 1` | Shows disk I/O per process. |
| `iostat -xz 1` | Shows disk utilization and latency. |
| `sar -u 1 5` | Shows CPU usage (sar also keeps history). |
| `sar -r` / `sar -n DEV` / `sar -d` | Shows memory / network / disk stats. |
| `sar -n TCP,ETCP 1` | Shows TCP connection and retransmission rates. |
| `sar -u -f /var/log/sa/sa26` | Shows CPU history for day 26 of the month. |
| `free -h` | Shows memory usage (watch the "available" column). |
| `grep -E 'MemAvailable\|Swap' /proc/meminfo` | Shows available memory and swap details. |
| `ps aux --sort=-rss \| head` | Shows processes using the most real memory. |
| `smem -rs rss` | Shows proportional memory use per process. |
| `dmesg -T \| grep -iE 'killed process\|out of memory'` | Finds OOM-killer events. |
| `journalctl -k \| grep -i oom` | Finds OOM events in the journal. |
| `cat /proc/PID/oom_score` | Shows how likely a process is to be OOM-killed. |
| `echo -1000 > /proc/PID/oom_score_adj` | Protects a process from the OOM killer. |
| `cat /sys/fs/cgroup/memory.max` | Shows a container's memory limit (cgroup v2). |
| `sync; echo 3 > /proc/sys/vm/drop_caches` | Drops filesystem caches (testing only). |
| `ps aux \| awk '$8 ~ /Z/'` | Lists zombie processes. |
| `ps -o ppid= -p ZPID` | Finds the parent of a zombie (restart the parent). |
| `ps -eo stat,pid,cmd \| awk '$1 ~ /D/'` | Lists processes stuck waiting on I/O. |

---

# Part 6 — Networking

## 32. Interfaces & Routing

| Command | What it does |
|---------|--------------|
| `ip a` | Shows all interfaces and their IP addresses. |
| `ip -br a` | Shows a brief one-line view per interface. |
| `ip -4 a show eth0` | Shows only IPv4 addresses on one interface. |
| `ip -c a` | Shows addresses with color. |
| `ip link` | Shows interfaces with MAC address, state and MTU. |
| `ip -s link show eth0` | Shows packet, error and drop counters. |
| `ip link set eth0 up` / `down` | Brings an interface up / down. |
| `ip link set eth0 mtu 9001` | Changes the interface MTU. |
| `ip addr add 10.0.1.50/24 dev eth0` | Adds a temporary IP address. |
| `ip addr del 10.0.1.50/24 dev eth0` | Removes an IP address. |
| `ip neigh` | Shows the ARP (neighbor) table. |
| `ip neigh flush dev eth0` | Clears the ARP table for an interface. |
| `ip route` / `ip r` | Shows the routing table. |
| `ip route get 8.8.8.8` | Shows which route, interface and source IP a destination uses. |
| `ip route add 10.20.0.0/16 via 10.0.1.1 dev eth0` | Adds a static route. |
| `ip route del 10.20.0.0/16` | Deletes a route. |
| `ip route add default via 10.0.1.1` | Sets the default gateway. |
| `ip rule` | Shows policy-routing rules. |
| `cat /sys/class/net/eth0/operstate` | Shows whether a link is up or down. |

## 33. Ports & Sockets

| Command | What it does |
|---------|--------------|
| `ss -tulnp` | Lists listening TCP/UDP ports with owning processes. |
| `ss -tlnp \| grep :443` | Checks what is listening on port 443. |
| `ss -tan` | Lists all TCP connections. |
| `ss -tan state established \| wc -l` | Counts established connections. |
| `ss -tan state time-wait \| wc -l` | Counts TIME_WAIT sockets (connection churn). |
| `ss -tan state close-wait` | Lists CLOSE_WAIT sockets (app not closing connections). |
| `ss -s` | Shows a summary of socket statistics. |
| `ss -tn dst 10.0.3.50` | Lists connections to a specific host. |
| `ss -tn sport = :8080` | Lists connections from a local port. |
| `ss -ti` | Shows TCP internals like RTT and retransmits. |
| `ss -x` | Lists Unix domain sockets. |
| `netstat -tulnp` | Legacy way to list listening ports. |
| `netstat -s \| grep -i retrans` | Shows TCP retransmission statistics. |
| `echo > /dev/tcp/host/443 && echo open` | Tests a TCP port using only bash. |

## 34. Connectivity Testing

| Command | What it does |
|---------|--------------|
| `ping -c 4 host` | Sends 4 ICMP packets to test reachability. |
| `ping -c 4 -W 2 host` | Pings with a 2-second timeout. |
| `ping -s 8972 -M do host` | Tests jumbo-frame MTU without fragmentation. |
| `traceroute host` | Shows each network hop to a host. |
| `traceroute -T -p 443 host` | Traces using TCP to get through firewalls. |
| `traceroute -n host` | Traces without resolving hop names. |
| `tracepath host` | Traces a path and discovers MTU without root. |
| `mtr -rw -c 100 host` | Reports packet loss and latency per hop. |
| `nc -zv host 5432` | Tests whether a TCP port is open. |
| `nc -zv -w 3 host 20-25` | Scans a port range with a 3-second timeout. |
| `nc -zvu host 53` | Tests a UDP port. |
| `nc -l 9000` | Listens on a port as a simple test server. |
| `telnet host 25` | Legacy way to test a TCP port. |
| `arping -I eth0 10.0.1.1` | Tests layer-2 reachability on the local network. |
| `iperf3 -s` / `iperf3 -c server -t 10` | Measures throughput between two hosts. |
| `nmap -Pn -p 22,80,443 host` | Scans specific ports (only with authorization). |

## 35. DNS

| Command | What it does |
|---------|--------------|
| `dig example.com` | Shows the full DNS answer for a name. |
| `dig +short example.com` | Prints only the resolved IPs. |
| `dig example.com MX` / `TXT` / `NS` / `CNAME` | Queries a specific record type. |
| `dig @8.8.8.8 example.com` | Queries a specific DNS server. |
| `dig +trace example.com` | Traces resolution from the root servers down. |
| `dig -x 10.0.1.15` | Does a reverse lookup of an IP. |
| `dig +noall +answer example.com` | Shows only the answer with TTLs. |
| `nslookup example.com 10.0.0.2` | Looks up a name using a given server. |
| `host example.com` | Simple DNS lookup. |
| `getent hosts example.com` | Resolves a name the same way applications do. |
| `resolvectl status` | Shows DNS servers per interface (systemd-resolved). |
| `resolvectl query example.com` | Resolves a name through systemd-resolved. |
| `resolvectl flush-caches` | Clears the local DNS cache. |
| `cat /etc/resolv.conf` | Shows configured DNS servers and search domains. |
| `cat /etc/hosts` | Shows static name-to-IP overrides. |

## 36. HTTP & TLS

| Command | What it does |
|---------|--------------|
| `curl URL` | Fetches a URL and prints the response. |
| `curl -I URL` | Fetches only the response headers. |
| `curl -v URL` | Shows DNS, connection, TLS and header details. |
| `curl -sS -o /dev/null -w "%{http_code}\n" URL` | Prints only the HTTP status code. |
| `curl -L URL` | Follows redirects. |
| `curl -k URL` | Ignores TLS certificate errors (testing only). |
| `curl -X POST -H "Content-Type: application/json" -d '{}' URL` | Sends a JSON POST request. |
| `curl -u user:pass URL` | Uses basic authentication. |
| `curl -H "Authorization: Bearer $T" URL` | Sends a bearer token header. |
| `curl --resolve host:443:10.0.1.15 https://host` | Tests a specific backend IP for a hostname. |
| `curl -H "Host: app.example.com" http://IP/` | Tests a virtual host on a specific server. |
| `curl --connect-timeout 5 -m 10 URL` | Sets connection and total timeouts. |
| `curl --retry 3 URL` | Retries a failed request. |
| `curl -x http://proxy:8080 URL` | Sends a request through a proxy. |
| `curl -O URL` | Downloads a file keeping its remote name. |
| `curl -w "dns:%{time_namelookup} conn:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n" -o /dev/null -s URL` | Breaks down where request latency is spent. |
| `curl --socks5-hostname localhost:1080 URL` | Sends a request through an SSH SOCKS proxy. |
| `wget URL` | Downloads a file. |
| `wget -c URL` | Resumes a partial download. |
| `wget -q -O - URL` | Prints a URL's content to stdout. |
| `openssl s_client -connect host:443 -servername host` | Tests a TLS handshake. |
| `openssl s_client ... \| openssl x509 -noout -dates` | Shows a remote certificate's expiry dates. |
| `openssl s_client -connect host:443 -showcerts` | Shows the full certificate chain. |
| `openssl x509 -in cert.pem -noout -text` | Shows full details of a certificate file. |
| `openssl x509 -in cert.pem -noout -enddate` | Shows when a certificate expires. |
| `openssl x509 -checkend 2592000 -noout -in cert.pem` | Checks whether a cert expires within 30 days. |
| `openssl verify -CAfile chain.pem cert.pem` | Validates a certificate against its chain. |
| `openssl req -new -newkey rsa:2048 -nodes -keyout k -out csr` | Creates a private key and certificate signing request. |
| `openssl rsa -noout -modulus -in key \| md5sum` | Fingerprints a key to check it matches a certificate. |

## 37. Network Configuration

| Command | What it does |
|---------|--------------|
| `nmcli device status` | Shows network devices and their state (RHEL). |
| `nmcli con show` | Lists connection profiles. |
| `nmcli con mod eth0 ipv4.addresses 10.0.1.50/24 ipv4.gateway 10.0.1.1 ipv4.dns "10.0.0.2" ipv4.method manual` | Sets a permanent static IP. |
| `nmcli con mod eth0 +ipv4.addresses 10.0.1.51/24` | Adds a secondary IP. |
| `nmcli con up eth0` | Applies and activates a connection. |
| `nmtui` | Opens a text menu for network settings. |
| `netplan try` | Applies Ubuntu network config with auto-rollback if connectivity breaks. |
| `netplan apply` | Applies Ubuntu network config permanently. |
| `hostnamectl set-hostname name` | Sets the hostname permanently. |

## 38. Firewalls

| Command | What it does |
|---------|--------------|
| `firewall-cmd --state` | Shows whether firewalld is running. |
| `firewall-cmd --get-active-zones` | Shows active zones and their interfaces. |
| `firewall-cmd --list-all` | Lists all rules in the default zone. |
| `firewall-cmd --permanent --add-service=https` | Permanently allows a named service. |
| `firewall-cmd --permanent --add-port=8080/tcp` | Permanently opens a port. |
| `firewall-cmd --permanent --remove-port=8080/tcp` | Permanently closes a port. |
| `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" port port="5432" protocol="tcp" accept'` | Allows a port only from a specific network. |
| `firewall-cmd --add-port=9000/tcp --timeout=1h` | Opens a port temporarily. |
| `firewall-cmd --reload` | Applies permanent rules. |
| `ufw status verbose` | Shows Ubuntu firewall rules. |
| `ufw allow 22/tcp` | Allows a port. |
| `ufw allow from 10.0.0.0/8 to any port 5432` | Allows a port only from a network. |
| `ufw deny 23` / `ufw delete allow 8080` | Blocks a port / removes a rule. |
| `ufw enable` | ⚠️ Turns on the firewall (allow SSH first). |
| `iptables -L -n -v --line-numbers` | Lists iptables rules with counters and numbers. |
| `iptables -t nat -L -n -v` | Lists NAT rules (Docker/Kubernetes). |
| `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` | Appends a rule allowing a port. |
| `iptables -I INPUT 1 -s 1.2.3.4 -j DROP` | Blocks an IP at the top of the chain. |
| `iptables -D INPUT 3` | Deletes rule number 3. |
| `iptables-save > file` / `iptables-restore < file` | Backs up / restores iptables rules. |
| `nft list ruleset` | Lists nftables rules. |

## 39. Packet Capture, Bandwidth & Kernel Tuning

| Command | What it does |
|---------|--------------|
| `tcpdump -i eth0` | Captures all packets on an interface. |
| `tcpdump -i any port 443 -nn` | Captures one port on all interfaces without name lookups. |
| `tcpdump -i eth0 host 10.0.3.50 and port 5432` | Captures traffic to one host and port. |
| `tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'` | Captures only SYN packets. |
| `tcpdump -i eth0 -c 100 -w cap.pcap` | Saves 100 packets to a file for Wireshark. |
| `tcpdump -r cap.pcap -nn` | Reads a saved capture. |
| `tcpdump -A port 80` | Prints packet contents as text. |
| `tcpdump -i any -nn udp port 53` | Captures DNS traffic. |
| `iftop -i eth0` | Shows live bandwidth per connection. |
| `nload eth0` | Shows incoming and outgoing throughput. |
| `nethogs` | Shows bandwidth per process. |
| `ethtool eth0` | Shows link speed, duplex and link status. |
| `ethtool -S eth0 \| grep -iE 'drop\|err'` | Shows NIC-level drops and errors. |
| `sysctl -a \| grep net.ipv4.tcp` | Lists TCP kernel settings. |
| `sysctl net.core.somaxconn` | Shows a single kernel setting. |
| `sysctl -w net.core.somaxconn=4096` | Changes a kernel setting until reboot. |
| `echo "key=value" > /etc/sysctl.d/99-tuning.conf` | Makes a kernel setting permanent. |
| `sysctl --system` | Reloads all sysctl config files. |
| `net.ipv4.ip_forward=1` | Enables packet forwarding (needed by Docker/Kubernetes). |
| `net.ipv4.ip_local_port_range` | Sets the range of outbound ephemeral ports. |
| `fs.file-max` | Sets the system-wide max open files. |
| `vm.swappiness=10` | Makes the kernel less eager to swap. |

---

# Part 7 — Disk & Storage

## 40. Disk Usage

| Command | What it does |
|---------|--------------|
| `df -h` | Shows free space per filesystem (first check on disk alerts). |
| `df -hT` | Adds the filesystem type column. |
| `df -h /var` | Shows which filesystem holds a path and how full it is. |
| `df -i` | Shows inode usage ("no space" with free blocks). |
| `df -h -x tmpfs -x devtmpfs` | Hides virtual filesystems. |
| `df -h -l` | Shows only local filesystems (avoids hanging NFS). |
| `du -sh /var/log` | Shows the total size of a directory. |
| `du -sh /var/* \| sort -hr \| head` | Shows the biggest items inside a directory. |
| `du -xh / --max-depth=1 \| sort -hr \| head` | Shows the biggest top-level directories on one filesystem. |
| `du -ah /var/log \| sort -hr \| head -20` | Shows the biggest files and directories. |
| `du --apparent-size -sh file` | Shows logical size (useful for sparse files). |
| `ncdu -x /` | Browses disk usage interactively and deletes files. |
| `find / -xdev -type f -size +500M -exec ls -lh {} \;` | Lists very large files. |
| `lsof +L1` | Finds deleted files still using disk space. |

## 41. Disks, Partitions & Filesystems

| Command | What it does |
|---------|--------------|
| `lsblk` | Shows disks, partitions, LVM and mount points as a tree. |
| `lsblk -f` | Adds filesystem type, label and UUID. |
| `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID` | Shows chosen columns. |
| `lsblk -d` | Shows whole disks only. |
| `blkid` | Shows UUIDs and filesystem types of devices. |
| `fdisk -l` | Lists partition tables of all disks. |
| `parted -l` | Lists partitions with parted. |
| `nvme list` | Lists NVMe devices (maps AWS EBS volume IDs). |
| `ls -l /dev/disk/by-uuid/` | Shows stable disk names by UUID. |
| `echo "- - -" > /sys/class/scsi_host/hostN/scan` | Detects a newly attached disk without reboot. |
| `echo 1 > /sys/class/block/sdb/device/rescan` | Detects a resized disk. |
| `fdisk /dev/sdb` | Partitions a disk interactively (n, t, p, w). |
| `gdisk /dev/sdb` | Partitions a GPT disk interactively. |
| `parted -s /dev/sdb mklabel gpt` | Creates a new GPT partition table. |
| `parted -s -a optimal /dev/sdb mkpart primary 0% 100%` | Creates one aligned partition using the whole disk. |
| `parted -s /dev/sdb set 1 lvm on` | Flags a partition for LVM. |
| `parted /dev/sdb resizepart 1 100%` | Grows a partition to the end of the disk. |
| `partprobe /dev/sdb` / `partx -u /dev/sdb` | Tells the kernel about partition changes. |
| `mkfs.xfs /dev/sdb1` | ⚠️ Creates an XFS filesystem (erases data). |
| `mkfs.xfs -f -L data /dev/sdb1` | Forces XFS creation and sets a label. |
| `mkfs.ext4 /dev/sdb1` | ⚠️ Creates an ext4 filesystem. |
| `mkfs.ext4 -L logs -m 1 /dev/sdb1` | Creates ext4 with a label and 1% root reserve. |
| `mkfs.ext4 -N 2000000 /dev/sdb1` | Creates ext4 with extra inodes for many small files. |
| `wipefs /dev/sdb` / `wipefs -a /dev/sdb` | Shows / ⚠️ erases old filesystem signatures. |
| `sgdisk --zap-all /dev/sdb` | ⚠️ Destroys GPT and MBR partition tables. |
| `blockdev --getsize64 /dev/sdb` | Shows exact device size in bytes. |
| `dd if=/dev/sda of=disk.img bs=4M status=progress` | Clones a disk to an image file. |
| `dd if=/dev/zero of=test bs=1M count=100` | Creates a 100 MB test file. |

## 42. Mounting & fstab

| Command | What it does |
|---------|--------------|
| `mount /dev/vg/lv /data` | Mounts a device on a directory. |
| `mount -t xfs /dev/sdb1 /data` | Mounts with an explicit filesystem type. |
| `mount UUID=xxxx /data` | Mounts by UUID. |
| `mount -o ro /dev/sdb1 /mnt` | Mounts read-only (recovery/forensics). |
| `mount -o remount,rw /` | Makes the root filesystem writable (rescue mode). |
| `mount -o remount,noexec /tmp` | Changes mount options without unmounting. |
| `mount -a` | Mounts everything in `/etc/fstab` (tests fstab changes). |
| `mount --bind /src /dst` | Makes a directory appear at another path. |
| `mount -o loop disk.iso /mnt` | Mounts an ISO or image file. |
| `mount \| column -t` | Lists current mounts in a table. |
| `findmnt` | Shows all mounts as a tree. |
| `findmnt /data` | Shows what is mounted at a path and its options. |
| `findmnt -T /var/log/app` | Shows which mount contains a path. |
| `findmnt --verify` | Validates `/etc/fstab` before rebooting. |
| `umount /data` | Unmounts a filesystem. |
| `umount -l /data` | Detaches now and cleans up when no longer busy. |
| `umount -f /mnt/nfs` | Force-unmounts a stale NFS share. |
| `UUID=xxxx /data xfs defaults,nofail 0 0` | fstab line that mounts a disk at boot without blocking boot. |
| `noatime` / `noexec,nosuid,nodev` | Mount options for speed / hardening. |
| `_netdev` | Mount option that waits for the network (NFS/iSCSI). |
| `cp /etc/fstab /etc/fstab.bak` | Backs up fstab before editing. |
| `systemctl daemon-reload` | Makes systemd pick up fstab changes. |

## 43. LVM

| Command | What it does |
|---------|--------------|
| `pvcreate /dev/sdb` | Prepares a disk for LVM. |
| `vgcreate vg_data /dev/sdb /dev/sdc` | Pools disks into a volume group. |
| `vgextend vg_data /dev/sdd` | Adds a disk to a volume group. |
| `lvcreate -n lv_app -L 50G vg_data` | Creates a 50 GB logical volume. |
| `lvcreate -n lv_logs -l 100%FREE vg_data` | Creates a logical volume using all free space. |
| `pvs` / `vgs` / `lvs` | Shows short summaries of PVs / VGs / LVs. |
| `pvdisplay` / `vgdisplay` / `lvdisplay` | Shows detailed PV / VG / LV info. |
| `lvs -a -o +devices` | Shows which disks each logical volume uses. |
| `lvextend -L +20G /dev/vg/lv` | Grows a logical volume by 20 GB. |
| `lvextend -l +100%FREE /dev/vg/lv` | Grows a logical volume into all free space. |
| `lvextend -r -L +20G /dev/vg/lv` | Grows the volume and its filesystem in one step. |
| `lvreduce -r -L 30G /dev/vg/lv` | ⚠️ Shrinks an ext4 volume and filesystem (unmount first). |
| `pvresize /dev/sdb1` | Updates a PV after its disk was enlarged. |
| `pvmove /dev/sdb` | Moves data off a disk while online. |
| `vgreduce vg_data /dev/sdb` | Removes a disk from a volume group. |
| `pvremove /dev/sdb` | Removes LVM labels from a disk. |
| `lvremove /dev/vg/lv` / `vgremove vg` | ⚠️ Deletes a logical volume / volume group. |
| `lvrename vg old new` | Renames a logical volume. |
| `lvcreate -s -n snap -L 5G /dev/vg/lv` | Takes a snapshot of a logical volume. |
| `mount -o ro,nouuid /dev/vg/snap /mnt/snap` | Mounts an XFS snapshot read-only. |
| `lvconvert --merge /dev/vg/snap` | Rolls a volume back to its snapshot. |

## 44. Growing Storage, Swap & Repair

| Command | What it does |
|---------|--------------|
| `aws ec2 modify-volume --volume-id vol-x --size 200` | Enlarges an AWS EBS volume. |
| `growpart /dev/nvme0n1 1` | Grows partition 1 to fill the enlarged disk. |
| `xfs_growfs /mountpoint` | Grows an XFS filesystem online (takes the mount point). |
| `resize2fs /dev/device` | Grows an ext4 filesystem online (takes the device). |
| `swapon --show` | Lists active swap. |
| `fallocate -l 4G /swapfile` | Creates a 4 GB file instantly. |
| `chmod 600 /swapfile && mkswap /swapfile` | Secures and formats the file as swap. |
| `swapon /swapfile` / `swapoff /swapfile` | Enables / disables swap. |
| `swapoff -a` | Disables all swap (required on Kubernetes nodes). |
| `sysctl vm.swappiness=10` | Reduces how eagerly the kernel swaps. |
| `e2fsck -f /dev/sdb1` | Force-checks an unmounted ext4 filesystem. |
| `e2fsck -fy /dev/sdb1` | Checks and auto-fixes an ext4 filesystem. |
| `fsck -n /dev/sdb1` | Checks without making changes. |
| `xfs_repair -n /dev/sdb1` | Dry-runs an XFS repair. |
| `xfs_repair /dev/sdb1` | Repairs an unmounted XFS filesystem. |
| `xfs_repair -L /dev/sdb1` | ⚠️ Clears the XFS log (last resort, may lose data). |
| `tune2fs -l /dev/sdb1` | Shows ext4 settings. |
| `tune2fs -m 1 /dev/sdb1` | Cuts root-reserved space to 1% (frees space instantly). |
| `tune2fs -L label` / `e2label dev label` | Sets an ext4 label. |
| `tune2fs -c 0 -i 0 /dev/sdb1` | Disables periodic forced checks. |
| `dumpe2fs -h /dev/sdb1` | Shows ext4 superblock info. |
| `xfs_info /data` | Shows XFS filesystem geometry. |
| `xfs_admin -L label /dev/sdb1` | Sets an XFS label. |
| `fstrim -av` | Trims unused blocks on SSDs. |

## 45. Disk Performance, RAID, Network Storage & Encryption

| Command | What it does |
|---------|--------------|
| `iostat -xz 1` | Shows per-disk IOPS, throughput, latency and utilization. |
| `iotop -oPa` | Shows which processes are doing disk I/O. |
| `pidstat -d 1` | Shows disk I/O per process. |
| `sar -d 1 5` | Shows disk activity (and history with `-f`). |
| `smartctl -H /dev/sda` | Shows a disk's overall health. |
| `smartctl -a /dev/sda` | Shows all SMART health attributes. |
| `nvme smart-log /dev/nvme0` | Shows NVMe drive health. |
| `fio --name=t --rw=randread --bs=4k --size=1G --iodepth=32 --runtime=60 --time_based` | Benchmarks random-read IOPS. |
| `hdparm -tT /dev/sda` | Quickly measures disk read speed. |
| `dd if=/dev/zero of=test bs=1M count=1024 oflag=direct` | Roughly measures disk write speed. |
| `cat /sys/block/sda/queue/scheduler` | Shows the disk I/O scheduler. |
| `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc` | Creates a software RAID 1 mirror. |
| `cat /proc/mdstat` | Shows RAID status and rebuild progress. |
| `mdadm --detail /dev/md0` | Shows RAID array details. |
| `mdadm /dev/md0 --fail /dev/sdb --remove /dev/sdb` | Marks and removes a failed disk. |
| `mdadm /dev/md0 --add /dev/sdd` | Adds a replacement disk. |
| `mdadm --detail --scan >> /etc/mdadm.conf` | Saves RAID config so it assembles at boot. |
| `showmount -e server` | Lists NFS shares a server exports. |
| `mount -t nfs4 -o hard,_netdev server:/export /mnt` | Mounts an NFS share. |
| `mount -t efs -o tls fs-xxx:/ /mnt/efs` | Mounts an AWS EFS filesystem. |
| `nfsstat -m` | Shows NFS mount options in use. |
| `exportfs -rav` | Reloads NFS server exports. |
| `mount -t cifs //server/share /mnt -o credentials=file` | Mounts a Windows/SMB share. |
| `iscsiadm -m discovery -t sendtargets -p IP` | Discovers iSCSI targets. |
| `iscsiadm -m node --login` | Logs in to iSCSI targets. |
| `setquota -u user 10G 12G 0 0 /home` | Sets a disk quota for a user (ext4). |
| `repquota -a` | Reports quota usage. |
| `xfs_quota -x -c 'report -h' /home` | Reports XFS quota usage. |
| `cryptsetup luksFormat /dev/sdb1` | ⚠️ Encrypts a partition with LUKS. |
| `cryptsetup open /dev/sdb1 name` | Unlocks an encrypted volume. |
| `cryptsetup close name` | Locks an encrypted volume. |
| `cryptsetup luksDump /dev/sdb1` | Shows LUKS header details. |

## 46. Containers & Kubernetes Storage

| Command | What it does |
|---------|--------------|
| `docker system df` | Shows disk used by images, containers and volumes. |
| `docker system df -v` | Shows detailed Docker disk usage. |
| `docker system prune -af --volumes` | ⚠️ Removes all unused Docker data. |
| `docker image prune -a --filter "until=168h"` | Removes unused images older than 7 days. |
| `docker builder prune -af` | Removes Docker build cache. |
| `find /var/lib/docker/containers -name "*-json.log" -size +500M` | Finds huge container log files. |
| `crictl images` / `crictl rmi --prune` | Lists / removes unused images on containerd nodes. |
| `kubectl get pv,pvc -A` | Lists persistent volumes and claims. |
| `kubectl describe pvc name -n ns` | Shows why a volume claim is pending or failing. |
| `kubectl get storageclass` | Lists available storage classes. |
| `kubectl describe node N \| grep -A5 Conditions` | Checks a node for disk pressure. |

---

# Part 8 — Quick Reference

## 47. Incident First-Response Commands

| Command | What it does |
|---------|--------------|
| `whoami && hostname && hostname -I` | Confirms you are on the right server as the right user. |
| `uptime` | Checks load and whether the box recently rebooted. |
| `dmesg -T \| tail -20` | Shows recent kernel errors (OOM, disk, network). |
| `systemctl --failed` | Lists failed services. |
| `journalctl -p err -S "-1h" --no-pager` | Shows all errors from the last hour. |
| `df -h && df -i` | Checks for full disks or exhausted inodes. |
| `free -h` | Checks memory pressure. |
| `vmstat 1 5` | Checks CPU saturation, I/O wait and swapping. |
| `top -o %CPU` | Finds the process using the most CPU. |
| `ps aux --sort=-rss \| head` | Finds the process using the most memory. |
| `iostat -xz 1` | Checks disk latency and saturation. |
| `ss -tulnp` | Confirms the service is listening on the expected port. |
| `curl -sv localhost:PORT/health` | Checks the application health endpoint locally. |
| `nc -zv host port` | Tests connectivity to a dependency. |
| `dig +short host` | Confirms DNS resolves correctly. |
| `lsof +L1` | Finds deleted files still holding disk space. |
| `last -x \| head` | Shows recent logins, reboots and shutdowns. |
| `rpm -qa --last \| head` / `tail /var/log/apt/history.log` | Shows what software changed recently. |
