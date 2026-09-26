# 05 · Archiving & Compression

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Package, compress, ship, and restore files — for backups, releases, log rotation, and moving data between servers.

---

## Table of Contents

1. [Archive vs Compression](#1-archive-vs-compression)
2. [Choosing a Format](#2-choosing-a-format)
3. [tar — The Core Tool](#3-tar--the-core-tool)
4. [gzip / gunzip / zcat](#4-gzip--gunzip--zcat)
5. [bzip2 / xz / zstd](#5-bzip2--xz--zstd)
6. [pigz — Parallel gzip](#6-pigz--parallel-gzip)
7. [zip / unzip](#7-zip--unzip)
8. [Other Formats: 7z, rar, cpio](#8-other-formats-7z-rar-cpio)
9. [Streaming Over SSH & Pipes](#9-streaming-over-ssh--pipes)
10. [Splitting & Verifying Archives](#10-splitting--verifying-archives)
11. [logrotate — Automated Log Compression](#11-logrotate--automated-log-compression)
12. [Real-World Scenarios](#12-real-world-scenarios)
13. [Interview Questions](#13-interview-questions)
14. [Cheat Sheet](#14-cheat-sheet)

---

## 1. Archive vs Compression

| Concept | What it does | Tool |
|---------|--------------|------|
| **Archive** | Bundles many files into ONE file (keeps perms, owners, structure) | `tar`, `cpio` |
| **Compression** | Makes ONE file smaller | `gzip`, `bzip2`, `xz`, `zstd` |
| **Both** | Archive + compress | `tar -czf`, `zip` |

> `gzip` alone compresses **single files** only. To compress a directory → `tar` it first.

---

## 2. Choosing a Format

| Format | Ext | Speed | Ratio | Use when |
|--------|-----|-------|-------|----------|
| gzip | `.gz`, `.tgz` | Fast | Good | Default everywhere, logs, releases |
| bzip2 | `.bz2` | Slow | Better | Legacy |
| xz | `.xz` | Slowest | Best | Distributing packages, cold storage |
| zstd | `.zst` | **Very fast** | Very good | Modern backups, big data ← recommended |
| zip | `.zip` | Fast | Good | Sharing with Windows, Lambda deploys |
| lz4 | `.lz4` | Fastest | Lower | Real-time, low CPU cost |

---

## 3. tar — The Core Tool

**Remember:** **c**reate · e**x**tract · lis**t** — **f**ile always last before name.

### Create

```bash
tar -cvf backup.tar /etc/nginx                 # Archive only
tar -czvf backup.tar.gz /etc/nginx             # + gzip  ← most common
tar -cjvf backup.tar.bz2 /data                 # + bzip2
tar -cJvf backup.tar.xz /data                  # + xz
tar --zstd -cvf backup.tar.zst /data           # + zstd
tar -czf app_$(date +%F).tar.gz -C /opt app    # Change dir first (clean paths)
tar -czf logs.tgz --exclude='*.tmp' --exclude='cache' /var/log/app
tar -czpf etc_backup.tgz /etc                  # Preserve permissions
tar -czf - /data | ssh backup 'cat > data.tgz' # Stream to remote
```

### List / inspect

```bash
tar -tvf backup.tar.gz                  # List contents (auto-detect compression)
tar -tzf backup.tar.gz | grep nginx.conf
```

### Extract

```bash
tar -xvf backup.tar                     # Extract here
tar -xzvf backup.tar.gz                 # Extract gzip
tar -xf backup.tar.xz                   # Modern tar auto-detects compression
tar -xzf backup.tgz -C /restore/        # Extract into target dir
tar -xzf backup.tgz etc/nginx/nginx.conf   # Extract ONE file
tar -xzf app.tgz --strip-components=1   # Drop top-level folder
tar -xzf backup.tgz --wildcards '*.conf'
```

### Update / compare

```bash
tar -rvf backup.tar newfile             # Append (uncompressed only)
tar -dvf backup.tar                     # Diff archive vs filesystem
```

| Option | What it does |
|--------|--------------|
| `-c` | Create a new archive file |
| `-x` | Extract files from the archive |
| `-t` | List archive contents, no extract |
| `-f` | Archive filename follows this option |
| `-v` | Verbose, list files being processed |
| `-z` | Compress/decompress with gzip |
| `-j` | Compress/decompress with bzip2 |
| `-J` | Compress/decompress with xz |
| `--zstd` | Compress/decompress with zstd |
| `-a` | Auto-pick compression from extension |
| `-C DIR` | Change to directory before acting |
| `-p` | Preserve original file permissions |
| `--exclude=PAT` | Skip files matching this pattern |
| `-X file` | Read exclude patterns from file |
| `-T file` | Read files to archive from list |
| `--strip-components=N` | Remove N leading path directories |
| `-r` | Append files to existing archive |
| `-u` | Append only newer files |
| `-d` | Compare archive against filesystem |
| `-k` | Don't overwrite existing files |
| `-P` | Keep leading `/` in paths |
| `--same-owner` | Keep original owners (root extract) |
| `--numeric-owner` | Use UID/GID numbers not names |
| `-I "cmd"` | Use custom compressor program |
| `--listed-incremental=snap` | Incremental backup using snapshot file |
| `--checkpoint=1000` | Show progress every 1000 records |

> ⚠️ **Tarbomb check:** always `tar -tf` before extracting an unknown archive — it may dump hundreds of files into your current dir or contain `../` paths.

### Incremental backups with tar

```bash
tar --listed-incremental=/backup/snap.file -czf /backup/full.tgz /data     # Level 0
tar --listed-incremental=/backup/snap.file -czf /backup/inc1.tgz /data     # Only changes
```

---

## 4. gzip / gunzip / zcat

```bash
gzip app.log                   # → app.log.gz (original removed)
gzip -k app.log                # Keep original
gzip -9 big.sql                # Max compression
gzip -1 big.sql                # Fastest
gzip -r /var/log/old/          # Every file in dir
gzip -l app.log.gz             # Show ratio
gzip -t app.log.gz             # Test integrity
gunzip app.log.gz              # Decompress (= gzip -d)
gzip -dc app.log.gz > app.log  # Decompress to stdout, keep .gz
zcat app.log.gz | grep ERROR   # Read without extracting
zless / zgrep / zdiff          # Pager / grep / diff on .gz
```

| Option | What it does |
|--------|--------------|
| `-d` | Decompress the file (like gunzip) |
| `-k` | Keep original file after compress |
| `-c` | Write output to stdout, keep files |
| `-r` | Recurse into directories, compress all |
| `-1` .. `-9` | Fastest (1) to best (9) |
| `-l` | List compressed/uncompressed size, ratio |
| `-t` | Test compressed file integrity |
| `-v` | Show name and compression percentage |
| `-f` | Force, overwrite existing output |

---

## 5. bzip2 / xz / zstd

```bash
bzip2 -k file ; bunzip2 file.bz2 ; bzcat file.bz2
xz -k -T0 file              # All CPU threads
xz -d file.xz ; unxz file.xz ; xzcat file.xz
xz -9e file                 # Extreme compression
zstd file                   # → file.zst (keeps original)
zstd -19 -T0 file           # High level, all threads
zstd -d file.zst ; unzstd file.zst ; zstdcat file.zst
zstd --rm file              # Remove source after
```

| Option | What it does |
|--------|--------------|
| `-k` | Keep original input file |
| `-d` | Decompress the given file |
| `-T0` | Use all available CPU threads |
| `-1..-19` | zstd compression level, higher=smaller |
| `--rm` | zstd: delete source after success |
| `-e` | xz extreme mode, slower, smaller |
| `-t` | Test integrity of compressed file |

---

## 6. pigz — Parallel gzip

Standard `gzip` uses 1 core. `pigz` uses all cores → 4–8× faster on big DB dumps.

```bash
pigz -p 8 big.sql
tar -I pigz -cf data.tgz /data          # tar with pigz
tar --use-compress-program="pigz -9" -cf data.tgz /data
unpigz big.sql.gz
```

| Option | What it does |
|--------|--------------|
| `-p N` | Use N parallel compression threads |
| `-k` | Keep the original input file |
| `-d` | Decompress instead of compress |

---

## 7. zip / unzip

```bash
zip archive.zip file1 file2
zip -r site.zip /var/www/html           # Recursive
zip -r lambda.zip . -x "*.git*" "tests/*"   # Exclude (AWS Lambda package)
zip -e secret.zip creds.txt             # Password protect
zip -9 -r best.zip dir/                 # Max compression
zip -u archive.zip newfile              # Update
unzip archive.zip
unzip archive.zip -d /opt/app/          # Extract to dir
unzip -l archive.zip                    # List contents
unzip -o archive.zip                    # Overwrite without prompt
unzip -q archive.zip                    # Quiet
unzip archive.zip "config/*"            # Extract subset
unzip -t archive.zip                    # Test integrity
```

| Option | What it does |
|--------|--------------|
| `zip -r` | Include directories recursively |
| `zip -x` | Exclude files matching pattern |
| `zip -e` | Encrypt with a password prompt |
| `zip -u` | Update changed or new files |
| `zip -q` | Quiet, no output messages |
| `unzip -d` | Extract into this target directory |
| `unzip -l` | List archive contents only |
| `unzip -o` | Overwrite existing files, no prompt |
| `unzip -n` | Never overwrite existing files |
| `unzip -t` | Test archive for errors |

---

## 8. Other Formats: 7z, rar, cpio

```bash
7z a archive.7z dir/            # Create
7z x archive.7z                 # Extract with paths
7z l archive.7z                 # List
unrar x file.rar                # Extract rar
find . -name "*.conf" | cpio -ov > confs.cpio   # cpio archive
cpio -idv < confs.cpio                          # Extract
rpm2cpio pkg.rpm | cpio -idmv                   # Unpack an RPM without installing
```

---

## 9. Streaming Over SSH & Pipes

No temp file needed — perfect when the source disk is full.

```bash
# Push a directory to remote, compressed on the wire
tar -czf - /var/www | ssh user@backup "cat > /backups/www_$(date +%F).tgz"

# Copy directory between servers, extract on arrival
tar -cf - /data | ssh user@newhost "tar -xf - -C /data"

# Pull remote logs to local, compressed
ssh web01 "tar -czf - /var/log/nginx" > web01_nginx_logs.tgz

# DB dump straight to compressed file
mysqldump -u root -p appdb | gzip > appdb_$(date +%F).sql.gz
pg_dump appdb | zstd -T0 > appdb.sql.zst

# Restore
gunzip < appdb.sql.gz | mysql -u root -p appdb
zstdcat appdb.sql.zst | psql appdb

# Upload to S3 without local copy
tar -czf - /data | aws s3 cp - s3://my-bucket/data_$(date +%F).tgz

# Progress bar with pv
tar -cf - /data | pv | gzip > data.tgz
```

---

## 10. Splitting & Verifying Archives

```bash
split -b 1G -d backup.tgz backup.tgz.part_      # 1GB parts
cat backup.tgz.part_* > backup.tgz              # Rejoin
sha256sum backup.tgz > backup.tgz.sha256        # Create checksum
sha256sum -c backup.tgz.sha256                  # Verify after transfer
gzip -t backup.tgz && tar -tzf backup.tgz >/dev/null && echo "Archive OK"
```

> **A backup you haven't test-restored is not a backup.**

---

## 11. logrotate — Automated Log Compression

Config: `/etc/logrotate.conf` and `/etc/logrotate.d/<app>`

```conf
/var/log/myapp/*.log {
    daily               # rotate every day
    rotate 14           # keep 14 old copies
    compress            # gzip rotated files
    delaycompress       # compress on the NEXT rotation
    missingok           # no error if log missing
    notifempty          # skip empty logs
    copytruncate        # copy then truncate (app keeps same FD)
    maxsize 500M        # rotate early if bigger
    dateext             # app.log-20260926 instead of app.log.1
    create 0640 app app # perms for new log
    sharedscripts
    postrotate
        systemctl reload myapp >/dev/null 2>&1 || true
    endscript
}
```

```bash
logrotate -d /etc/logrotate.d/myapp     # Dry run / debug
logrotate -f /etc/logrotate.d/myapp     # Force rotation now
logrotate -v /etc/logrotate.conf        # Verbose
cat /var/lib/logrotate/logrotate.status # Last rotation times
```

| Option | What it does |
|--------|--------------|
| `-d` | Debug dry run, change nothing |
| `-f` | Force rotation even if not due |
| `-v` | Verbose, show rotation decisions |

---

## 12. Real-World Scenarios

**Before a risky change — snapshot config**
```bash
sudo tar -czpf /root/etc_$(hostname)_$(date +%F_%H%M).tgz /etc
```

**Release artifact in CI/CD**
```bash
tar -czf myapp-${VERSION}.tar.gz -C build .
sha256sum myapp-${VERSION}.tar.gz > myapp-${VERSION}.tar.gz.sha256
```

**Deploy on server**
```bash
mkdir -p /opt/myapp/releases/${VERSION}
tar -xzf myapp-${VERSION}.tar.gz -C /opt/myapp/releases/${VERSION}
ln -sfn /opt/myapp/releases/${VERSION} /opt/myapp/current
```

**Compress old logs to free space**
```bash
find /var/log/app -name "*.log" -mtime +3 ! -name "*.gz" -exec gzip {} +
```

**Collect diagnostics bundle for vendor / RCA**
```bash
tar -czf diag_$(hostname)_$(date +%F).tgz /var/log/messages /var/log/app /etc/app 2>/dev/null
```

---

## 13. Interview Questions

1. **Archive vs compression?** Archive bundles files; compression shrinks them.
2. **Extract one file from a tarball?** `tar -xzf a.tgz path/to/file`
3. **gzip vs xz vs zstd?** gzip balanced; xz smallest but slow; zstd fast + great ratio.
4. **Why `-C` in tar?** Avoid absolute paths; control where files go.
5. **Copy a directory to another server without temp space?** `tar -cf - dir | ssh host "tar -xf - -C /dst"`
6. **`copytruncate` in logrotate — why?** App keeps writing to same file handle without restart.
7. **Speed up gzip on multi-core?** `pigz`.

---

## 14. Cheat Sheet

```bash
tar -czvf out.tgz dir      # create gz
tar -cJvf out.txz dir      # create xz
tar --zstd -cf out.tzst d  # create zstd
tar -tzvf out.tgz          # list
tar -xzvf out.tgz -C /dst  # extract
tar -xzf a.tgz --strip-components=1
gzip -k -9 | gunzip | zcat | zgrep | gzip -t
xz -T0 -k | zstd -T0 -19 | pigz -p 8
zip -r a.zip d -x "*.git*" | unzip -l | unzip -d /dst
tar -czf - d | ssh h "cat > d.tgz"
sha256sum -c | logrotate -d / -f
```
