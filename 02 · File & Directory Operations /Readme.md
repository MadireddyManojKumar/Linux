# 02 · File & Directory Operations (CRUD)

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Create, Read, Update, Delete files and directories safely — plus find, link, compare and sync them like you would on production servers.

---

## Table of Contents

1. [CREATE — Files & Directories](#1-create--files--directories)
2. [READ — Viewing Files](#2-read--viewing-files)
3. [UPDATE — Copy, Move, Rename, Edit](#3-update--copy-move-rename-edit)
4. [DELETE — Removing Safely](#4-delete--removing-safely)
5. [Links — Hard vs Soft](#5-links--hard-vs-soft)
6. [File Info — `stat`, `file`, `du`](#6-file-info--stat-file-du)
7. [FIND — The SRE Power Tool](#7-find--the-sre-power-tool)
8. [xargs — Act on Many Files](#8-xargs--act-on-many-files)
9. [Compare & Verify Files](#9-compare--verify-files)
10. [rsync — Copy Like a Pro](#10-rsync--copy-like-a-pro)
11. [Wildcards / Globbing](#11-wildcards--globbing)
12. [Real-World Scenarios](#12-real-world-scenarios)
13. [Interview Questions](#13-interview-questions)
14. [Cheat Sheet](#14-cheat-sheet)

---

## 1. CREATE — Files & Directories

### `touch` — Create empty file / update timestamp

```bash
touch app.log                    # Create empty file
touch file{1..5}.txt             # file1.txt ... file5.txt
touch -d "2026-01-01" old.txt    # Set specific timestamp
touch -r ref.txt new.txt         # Copy timestamp from ref
```

| Option | What it does |
|--------|--------------|
| `-a` | Change only the access time |
| `-m` | Change only the modification time |
| `-c` | Don't create file if missing |
| `-d "date"` | Set timestamp to given date |
| `-r ref` | Use timestamp from reference file |

### `mkdir` — Make directories

```bash
mkdir releases
mkdir -p /opt/app/{bin,conf,logs,data}   # Full tree in one shot
mkdir -m 750 /opt/secure                 # With permissions
mkdir -pv /srv/www/site/{css,js,img}
```

| Option | What it does |
|--------|--------------|
| `-p` | Create parents, no error if exists |
| `-m 750` | Set permissions while creating dir |
| `-v` | Print each directory as created |

### Create files with content

```bash
echo "hello" > file.txt           # Overwrite
echo "line2" >> file.txt          # Append
printf "a\nb\n" > list.txt        # Formatted output
cat > notes.txt                   # Type, then Ctrl+D
cat <<'EOF' > /etc/app/app.conf   # Heredoc ('EOF' = no variable expansion)
PORT=8080
ENV=prod
EOF
sudo tee /etc/sysctl.d/99-app.conf <<< "vm.swappiness=10"   # Write as root
```

> **Why `sudo tee`?** `sudo echo x > /etc/file` fails — the redirect runs as *you*, not root. `tee` runs as root.

### `install` — copy + set perms + owner in one step

```bash
sudo install -m 0755 -o root -g root ./mytool /usr/local/bin/mytool
sudo install -d -m 0750 -o app -g app /var/lib/app
```

| Option | What it does |
|--------|--------------|
| `-m` | Set permission mode on target |
| `-o` / `-g` | Set owner / group on target |
| `-d` | Create directories instead of copying |

### `mktemp` — safe temp files for scripts

```bash
TMP=$(mktemp)                 # /tmp/tmp.X8a2b...
TMPDIR=$(mktemp -d)           # Temp directory
trap 'rm -rf "$TMPDIR"' EXIT  # Auto-cleanup on script exit
```

---

## 2. READ — Viewing Files

| Command | Best for |
|---------|----------|
| `cat` | Small files, dump everything |
| `less` | Big files, scroll & search ← preferred |
| `head` | First N lines |
| `tail` | Last N lines / live logs |
| `more` | Old pager, forward only |
| `tac` | Print file in reverse |
| `nl` | Print with line numbers |
| `wc` | Count lines/words/bytes |

### `cat`

```bash
cat file.txt
cat -n file.txt            # Numbered lines
cat -A file.txt            # Reveal tabs (^I), line ends ($), CRLF (^M)
cat a.txt b.txt > all.txt  # Concatenate
```

| Option | What it does |
|--------|--------------|
| `-n` | Number all output lines |
| `-b` | Number only non-blank lines |
| `-A` | Show hidden chars: tabs, endings |
| `-s` | Squeeze repeated blank lines into one |

> `cat -A` is how you catch Windows `\r\n` line endings breaking a bash script (`^M$`).

### `less`

```bash
less /var/log/syslog
less +F /var/log/nginx/access.log   # Follow mode like tail -f
less -N file                        # Line numbers
less -S file                        # Don't wrap long lines
```

**Keys:** `/text` search forward · `?text` backward · `n`/`N` next/prev · `G` end · `g` top · `F` follow · `q` quit

### `head` / `tail`

```bash
head -n 20 file.log
head -c 100 file.bin             # First 100 bytes
tail -n 50 app.log
tail -f app.log                  # Follow live ← incident #1 tool
tail -F app.log                  # Follow even across log rotation
tail -f app.log | grep --line-buffered ERROR
tail -n +10 file                 # From line 10 to end
tail -f a.log b.log              # Follow multiple files
```

| Option | What it does |
|--------|--------------|
| `-n N` | Show N lines |
| `-c N` | Show N bytes |
| `-f` | Follow file as it grows |
| `-F` | Follow by name, survives rotation |
| `-n +N` | Start output from line N |
| `--pid=PID` | Stop following when process dies |

### `wc`

```bash
wc -l access.log                 # Line count
ls | wc -l                       # Number of files
```

| Option | What it does |
|--------|--------------|
| `-l` | Count lines only |
| `-w` | Count words only |
| `-c` | Count bytes only |
| `-m` | Count characters (multibyte aware) |

---

## 3. UPDATE — Copy, Move, Rename, Edit

### `cp` — Copy

```bash
cp app.conf app.conf.bak                 # Backup before edit ← always
cp app.conf{,.bak.$(date +%F)}           # app.conf.bak.2026-09-26
cp -r src/ dest/                         # Copy directory
cp -a /var/www /backup/                  # Archive: keep perms/owner/times/links
cp -iv *.conf /etc/app/                  # Interactive + verbose
cp -u src/* dest/                        # Only newer files
```

| Option | What it does |
|--------|--------------|
| `-r` / `-R` | Copy directories recursively |
| `-a` | Archive: preserve everything, recursive |
| `-p` | Preserve mode, owner, timestamps |
| `-i` | Ask before overwriting existing files |
| `-n` | Never overwrite existing files |
| `-f` | Force overwrite, remove if needed |
| `-u` | Copy only when source is newer |
| `-v` | Print each file being copied |
| `-l` | Hard link instead of copy |
| `-s` | Symlink instead of copy |
| `--backup=numbered` | Keep numbered backups of overwritten |

### `mv` — Move / Rename

```bash
mv old.txt new.txt                   # Rename
mv app.log /var/log/archive/         # Move
mv -i *.log archive/                 # Prompt before overwrite
mv -n src dest                       # Never overwrite
mv -v release-1.2 current            # Verbose
```

| Option | What it does |
|--------|--------------|
| `-i` | Prompt before overwriting any file |
| `-n` | Don't overwrite existing destination |
| `-f` | Overwrite without asking at all |
| `-v` | Show what is being moved |
| `-u` | Move only if source newer |
| `-b` | Backup destination before overwrite |

### `rename` — bulk rename

```bash
rename 's/\.log$/.log.old/' *.log        # Perl rename (Debian/Ubuntu)
rename .log .log.old *.log               # util-linux rename (RHEL)
for f in *.txt; do mv "$f" "${f%.txt}.md"; done   # Portable
```

### In-place edit (non-interactive)

```bash
sed -i.bak 's/8080/9090/g' app.conf      # Replace, keep .bak copy
```

> Full `sed` coverage in **04 · Text Processing**.

---

## 4. DELETE — Removing Safely

### `rm`

```bash
rm file.txt
rm -i *.log                      # Confirm each
rm -r old_dir/                   # Directory + contents
rm -rf /opt/app/tmp/*            # Force, no prompts ⚠️
rm -- -weirdname                 # File starting with dash
rm -I *                          # Prompt once if > 3 files
```

| Option | What it does |
|--------|--------------|
| `-r` | Remove directories and their contents |
| `-f` | Force, ignore missing, no prompts |
| `-i` | Prompt before every single removal |
| `-I` | Prompt once for bulk deletes |
| `-v` | Print each file being removed |
| `-d` | Remove empty directories too |
| `--preserve-root` | Refuse to delete `/` (default) |

### `rmdir`

```bash
rmdir empty_dir
rmdir -p a/b/c          # Remove chain of empty dirs
```

### `shred` — secure delete

```bash
shred -u -z -n 3 secrets.txt
```

| Option | What it does |
|--------|--------------|
| `-n 3` | Overwrite file three times |
| `-z` | Final pass with zeros, hides shredding |
| `-u` | Delete file after overwriting it |

### ⚠️ Production safety rules

```bash
# NEVER do this — if $DIR is empty it becomes rm -rf /
rm -rf $DIR/*

# DO this — fails if DIR unset/empty
rm -rf "${DIR:?DIR is not set}"/*

# Preview before deleting
find /var/log/app -name "*.log" -mtime +30 -print     # look first
find /var/log/app -name "*.log" -mtime +30 -delete    # then delete
```

> **Deleted file still using disk?** A process still holds it open. Find it: `lsof +L1` or `lsof | grep deleted`. Fix: restart process, or truncate: `: > /proc/PID/fd/N`.

### Truncate (empty) a file without deleting it

```bash
> app.log                 # Empty it (keeps inode, process keeps writing)
truncate -s 0 app.log     # Same, explicit
: > app.log               # Same, POSIX safe
```

> Use truncate instead of `rm` for live log files — `rm` won't free space while the app has it open.

---

## 5. Links — Hard vs Soft

```bash
ln -s /opt/app/releases/v2.1 /opt/app/current   # Symlink
ln -sfn /opt/app/releases/v2.2 /opt/app/current # Atomic-ish switch (deploys)
ln original.txt hardlink.txt                     # Hard link
readlink -f /opt/app/current                     # Resolve final target
ls -l /opt/app/current                           # current -> releases/v2.2
find / -xdev -samefile original.txt              # All hard links to file
```

| Option | What it does |
|--------|--------------|
| `-s` | Create symbolic (soft) link |
| `-f` | Remove existing destination link first |
| `-n` | Treat link-to-dir as normal file |
| `-v` | Print name of each link |
| `readlink -f` | Follow all links to real path |

| Feature | Hard Link | Soft (Symbolic) Link |
|---------|-----------|----------------------|
| Points to | Inode (data) | Path (name) |
| Cross filesystem | ❌ No | ✅ Yes |
| Link to directory | ❌ No | ✅ Yes |
| Original deleted | Still works | Broken (dangling) |
| Own inode | Same inode | Different inode |

> **Real world:** Blue/green & Capistrano-style deploys switch a `current` symlink between release directories — instant rollback.

```bash
find /etc -xtype l          # Find broken symlinks
```

---

## 6. File Info — `stat`, `file`, `du`

```bash
stat app.log                       # Size, inode, perms, atime/mtime/ctime
stat -c '%a %U:%G %n' /etc/shadow  # 640 root:shadow /etc/shadow
file backup.tar.gz                 # Real file type (ignores extension)
file -i data.bin                   # MIME type
du -sh /var/log                    # Total dir size
du -h --max-depth=1 /var | sort -hr | head   # Biggest subdirs
```

| Option | What it does |
|--------|--------------|
| `stat -c FORMAT` | Custom output: `%a` perms, `%s` size |
| `du -s` | Summary total only, not each |
| `du -h` | Human-readable sizes |
| `du --max-depth=1` | Only go one level deep |
| `du -x` | Stay on one filesystem only |

| Timestamp | Changes when |
|-----------|--------------|
| **atime** | File was read |
| **mtime** | Content modified |
| **ctime** | Metadata changed (perms, owner, rename) |

---

## 7. FIND — The SRE Power Tool

```bash
find <where> <tests> <actions>
```

### By name / type

```bash
find /etc -name "nginx.conf"
find / -iname "*.PEM" 2>/dev/null        # Case-insensitive, hide errors
find /var/log -type f -name "*.log"
find /opt -type d -name "node_modules"
find . -type l                            # Symlinks
find . -maxdepth 1 -type f                # Current dir only
```

### By size / time

```bash
find / -xdev -type f -size +500M 2>/dev/null     # Big files ← disk full
find /var/log -mtime +30                         # Modified > 30 days ago
find /tmp -mmin -10                              # Modified in last 10 min
find . -newer deploy.marker                      # Newer than a file
find /data -type f -empty                        # Empty files
```

### By owner / perms

```bash
find /home -user deploy
find / -nouser -o -nogroup 2>/dev/null           # Orphaned files (after offboarding)
find / -perm -4000 -type f 2>/dev/null           # SUID binaries (security audit)
find /var/www -type f -perm 0777                 # World-writable files
```

### Actions

```bash
find /var/log/app -name "*.log" -mtime +7 -delete
find . -name "*.sh" -exec chmod +x {} \;          # One exec per file
find . -name "*.log" -exec gzip {} +              # Batch (faster)
find /etc -name "*.conf" -exec grep -l "8080" {} +
find . -type f -print0 | xargs -0 ls -l           # Safe with spaces
```

| Option | What it does |
|--------|--------------|
| `-name` | Match filename, case sensitive |
| `-iname` | Match filename, ignore case |
| `-type f/d/l` | File, directory, or symlink |
| `-size +100M` | Larger than 100 megabytes |
| `-size -1k` | Smaller than one kilobyte |
| `-mtime +7` | Modified more than 7 days ago |
| `-mtime -1` | Modified within last 24 hours |
| `-mmin -10` | Modified within last 10 minutes |
| `-user` / `-group` | Owned by user or group |
| `-perm -4000` | Has SUID bit set |
| `-nouser` | No valid owner (deleted user) |
| `-empty` | Empty file or empty directory |
| `-maxdepth N` | Descend at most N levels |
| `-mindepth N` | Skip first N levels |
| `-xdev` | Don't cross into other filesystems |
| `-newer file` | Modified after given file |
| `-delete` | Delete each matched file |
| `-exec cmd {} \;` | Run command once per match |
| `-exec cmd {} +` | Run command with many matches |
| `-print0` | Null-separated output, safe names |
| `-not` / `!` | Negate the next test |
| `-o` | Logical OR between tests |
| `-prune` | Skip this directory entirely |

```bash
# Exclude a directory
find . -path ./node_modules -prune -o -name "*.js" -print
```

---

## 8. xargs — Act on Many Files

```bash
cat hosts.txt | xargs -I{} ssh {} uptime
find . -name "*.tmp" -print0 | xargs -0 rm -f
echo "a b c" | xargs -n1                        # One per line
cat urls.txt | xargs -P 8 -n 1 curl -sO          # 8 parallel downloads
```

| Option | What it does |
|--------|--------------|
| `-0` | Input is null-separated (with -print0) |
| `-n N` | Use N arguments per command |
| `-I{}` | Replace `{}` with each input |
| `-P N` | Run N processes in parallel |
| `-r` | Don't run if input empty |
| `-t` | Print each command before running |

---

## 9. Compare & Verify Files

```bash
diff old.conf new.conf
diff -u old.conf new.conf          # Unified (git-style) ← most readable
diff -r dir1/ dir2/                # Compare directories
diff <(ssh web1 cat /etc/app.conf) <(ssh web2 cat /etc/app.conf)  # Config drift!
cmp file1 file2                    # Byte compare (binaries)
sha256sum app.tar.gz               # Checksum
sha256sum -c app.tar.gz.sha256     # Verify download
md5sum file
```

| Option | What it does |
|--------|--------------|
| `diff -u` | Unified format with context lines |
| `diff -r` | Recursively compare two directories |
| `diff -q` | Only report whether files differ |
| `diff -y` | Side-by-side two column view |
| `diff -w` | Ignore all whitespace differences |
| `sha256sum -c` | Verify checksums listed in file |

---

## 10. rsync — Copy Like a Pro

```bash
rsync -avh src/ dest/                              # Local sync
rsync -avhz --progress /data/ user@backup:/data/   # Remote over SSH
rsync -avh --delete src/ dest/                     # Mirror (delete extras) ⚠️
rsync -avhn --delete src/ dest/                    # Dry run first ← always
rsync -avh --exclude='*.log' --exclude='.git' src/ dest/
rsync -avh -e "ssh -p 2222 -i ~/.ssh/key" src/ host:/dst/
rsync -avhP bigfile host:/tmp/                     # Resume partial transfers
```

| Option | What it does |
|--------|--------------|
| `-a` | Archive: recursive, keep perms, times |
| `-v` | Verbose, list transferred files |
| `-h` | Human-readable numbers in output |
| `-z` | Compress data during transfer |
| `-n` | Dry run, change nothing |
| `-P` | Progress bar plus resume partial |
| `--delete` | Delete dest files not in source |
| `--exclude` | Skip files matching this pattern |
| `-e ssh` | Specify remote shell and options |
| `--bwlimit=5000` | Limit bandwidth to KB per second |
| `-u` | Skip files newer on destination |
| `-c` | Compare by checksum, not time |

> **Trailing slash matters:** `src/` = contents of src. `src` = the folder itself into dest.

### `scp` (simple, legacy)

```bash
scp file.txt user@host:/tmp/
scp -r dir/ user@host:/opt/
scp -P 2222 -i key.pem file user@host:~
```

---

## 11. Wildcards / Globbing

| Pattern | Matches |
|---------|---------|
| `*` | Any number of characters |
| `?` | Exactly one character |
| `[abc]` | One of a, b, c |
| `[0-9]` | One digit |
| `[!a]` | Anything except a |
| `{a,b}` | Brace expansion: a and b |
| `{1..10}` | Sequence 1 to 10 |
| `**` | Recursive (with `shopt -s globstar`) |

```bash
ls app-*.log
rm log.?
cp config.{yml,yml.bak}
mkdir -p env/{dev,qa,prod}
```

---

## 12. Real-World Scenarios

**Safe config change**
```bash
cp -p /etc/nginx/nginx.conf{,.bak.$(date +%F)}
vim /etc/nginx/nginx.conf
nginx -t && systemctl reload nginx || cp /etc/nginx/nginx.conf.bak.$(date +%F) /etc/nginx/nginx.conf
```

**Log cleanup: older than 14 days**
```bash
find /var/log/app -type f -name "*.log*" -mtime +14 -print -delete
```

**Find what's eating disk**
```bash
du -xh / --max-depth=1 2>/dev/null | sort -hr | head
find / -xdev -type f -size +1G -exec ls -lh {} \; 2>/dev/null
```

**Zero-downtime release switch**
```bash
ln -sfn /opt/app/releases/2026-09-26 /opt/app/current && systemctl reload app
```

---

## 13. Interview Questions

1. **`cp -a` vs `cp -r`?** `-a` preserves perms/owner/time/links; `-r` just copies.
2. **Hard vs soft link?** Hard = same inode, same FS; soft = path pointer, can cross FS.
3. **Deleted a 10G log but `df` shows no space freed — why?** Process still holds the file open; `lsof +L1`, restart or truncate.
4. **`-exec {} \;` vs `-exec {} +`?** `\;` per file; `+` batches — faster.
5. **Find files modified in last 24h?** `find / -mtime -1`
6. **mtime vs ctime?** mtime = content changed; ctime = metadata changed.
7. **Why dry-run rsync with `--delete`?** It removes files on destination — mistakes are destructive.

---

## 14. Cheat Sheet

```bash
touch | mkdir -p | install -m -o -g | mktemp | tee
cat -A | less +F | head -n | tail -F | wc -l
cp -a | cp file{,.bak} | mv -i | rename
rm -rf "${DIR:?}" | rmdir | shred -uz | truncate -s 0
ln -sfn | readlink -f | find -xtype l
stat | file | du -sh
find / -xdev -size +1G | -mtime +30 -delete | -perm -4000 | -exec {} +
xargs -0 -P -I{}
diff -u | sha256sum -c
rsync -avhzP --delete -n
```
