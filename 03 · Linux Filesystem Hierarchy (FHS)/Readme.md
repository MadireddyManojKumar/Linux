# 03 · Linux Filesystem Hierarchy (FHS)

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Know exactly where configs, logs, binaries, data and runtime state live — so you can troubleshoot any Linux server without guessing.

---

## Table of Contents

1. [The Big Picture](#1-the-big-picture)
2. [Directory-by-Directory Reference](#2-directory-by-directory-reference)
3. [/etc — Configuration Files You Must Know](#3-etc--configuration-files-you-must-know)
4. [/var — Logs, Caches, State](#4-var--logs-caches-state)
5. [/proc — Live Kernel & Process Data](#5-proc--live-kernel--process-data)
6. [/sys — Hardware & Kernel Tunables](#6-sys--hardware--kernel-tunables)
7. [/dev — Device Files](#7-dev--device-files)
8. [Binaries: /bin, /sbin, /usr, /usr/local, /opt](#8-binaries-bin-sbin-usr-usrlocal-opt)
9. [Temp & Runtime: /tmp, /var/tmp, /run](#9-temp--runtime-tmp-vartmp-run)
10. [Commands to Explore the Hierarchy](#10-commands-to-explore-the-hierarchy)
11. [Containers & Cloud Paths](#11-containers--cloud-paths)
12. [Real-World: Where Do I Look When…](#12-real-world-where-do-i-look-when)
13. [Interview Questions](#13-interview-questions)
14. [Cheat Sheet](#14-cheat-sheet)

---

## 1. The Big Picture

Linux has **one tree** starting at `/` (root). Disks, USBs, NFS shares are **mounted** into this tree — there are no `C:` / `D:` drives.

```
/
├── bin  -> usr/bin      Essential user commands
├── sbin -> usr/sbin     Essential admin commands
├── boot                 Kernel, initramfs, GRUB
├── dev                  Device files (disks, tty, null)
├── etc                  System-wide configuration
├── home                 Regular users' home dirs
├── lib  -> usr/lib      Shared libraries, kernel modules
├── media                Auto-mounted removable media
├── mnt                  Temporary manual mounts
├── opt                  Third-party / vendor software
├── proc                 Virtual: processes & kernel info
├── root                 Root user's home
├── run                  Runtime data since boot (PIDs, sockets)
├── srv                  Data served by the system (web, ftp)
├── sys                  Virtual: devices & kernel tunables
├── tmp                  Temp files (cleared on reboot)
├── usr                  User programs, libs, docs (read-only-ish)
└── var                  Variable data: logs, spool, cache, DBs
```

> **Modern distros ("usr merge"):** `/bin`, `/sbin`, `/lib` are symlinks into `/usr`. Check with `ls -l /bin`.

---

## 2. Directory-by-Directory Reference

| Directory | What lives there | SRE relevance |
|-----------|------------------|---------------|
| `/` | Root of everything | Must never fill up |
| `/bin` | `ls`, `cp`, `bash`, `cat` | Core tools |
| `/sbin` | `fdisk`, `ip`, `mkfs`, `reboot` | Admin tools |
| `/boot` | `vmlinuz-*`, `initramfs-*`, `grub2/` | Full `/boot` = failed kernel updates |
| `/dev` | `sda`, `nvme0n1`, `null`, `zero`, `random` | Disks & devices |
| `/etc` | All configuration | Most edits happen here |
| `/home` | `/home/<user>` | User data, dotfiles, SSH keys |
| `/lib`, `/lib64` | `.so` libs, `/lib/modules/<kernel>` | Library & driver issues |
| `/media` | USB/CD auto-mounts | Rare on servers |
| `/mnt` | Manual temp mounts | Rescue & migrations |
| `/opt` | Vendor apps (`/opt/splunk`, `/opt/app`) | Custom installs |
| `/proc` | Virtual, per-process info | Live debugging |
| `/root` | Root's home | Not `/` ! |
| `/run` | PID files, sockets, `/run/user/UID` | tmpfs, gone at reboot |
| `/srv` | Site data (`/srv/www`) | Service data |
| `/sys` | Devices, drivers, tunables | Hardware & cgroups |
| `/tmp` | Short-lived temp files | Often tmpfs, sticky bit |
| `/usr` | `bin`, `lib`, `share`, `local` | Packaged software |
| `/usr/local` | Manually built software | Not touched by package manager |
| `/var` | Logs, spool, cache, lib | Most common "disk full" culprit |
| `/lost+found` | Fragments recovered by `fsck` | Present on ext filesystems |

---

## 3. /etc — Configuration Files You Must Know

### Users & Auth

| File | Purpose |
|------|---------|
| `/etc/passwd` | User accounts (no passwords) |
| `/etc/shadow` | Hashed passwords + aging (root only) |
| `/etc/group` | Groups and members |
| `/etc/gshadow` | Group passwords/admins |
| `/etc/sudoers`, `/etc/sudoers.d/` | Sudo rules (edit with `visudo`) |
| `/etc/login.defs` | Default UID ranges, password aging |
| `/etc/skel/` | Template copied to new home dirs |
| `/etc/pam.d/` | PAM authentication modules |
| `/etc/security/limits.conf` | ulimits (open files, processes) |

### Networking

| File | Purpose |
|------|---------|
| `/etc/hostname` | System hostname |
| `/etc/hosts` | Static name → IP map (checked before DNS) |
| `/etc/resolv.conf` | DNS servers & search domains |
| `/etc/nsswitch.conf` | Lookup order (files, dns, ldap) |
| `/etc/netplan/*.yaml` | Network config (Ubuntu) |
| `/etc/NetworkManager/system-connections/` | Network config (RHEL 9+) |
| `/etc/sysconfig/network-scripts/` | Legacy RHEL network config |
| `/etc/ssh/sshd_config` | SSH server config |
| `/etc/ssh/ssh_config` | SSH client defaults |
| `/etc/services` | Port number ↔ service names |

### System & Boot

| File | Purpose |
|------|---------|
| `/etc/fstab` | Filesystems mounted at boot ⚠️ typo = unbootable |
| `/etc/os-release` | Distro name & version |
| `/etc/systemd/system/` | Custom & override unit files |
| `/etc/sysctl.conf`, `/etc/sysctl.d/` | Kernel parameters |
| `/etc/crontab`, `/etc/cron.d/` | System cron jobs |
| `/etc/logrotate.conf`, `/etc/logrotate.d/` | Log rotation rules |
| `/etc/rsyslog.conf` | Syslog routing |
| `/etc/default/grub` | Kernel boot parameters |
| `/etc/timezone`, `/etc/localtime` | Time zone |
| `/etc/environment` | System-wide env vars |
| `/etc/profile`, `/etc/profile.d/` | Login shell setup, all users |
| `/etc/motd` | Message shown after login |

### Packages

| File | Purpose |
|------|---------|
| `/etc/apt/sources.list`, `/etc/apt/sources.list.d/` | APT repos (Debian/Ubuntu) |
| `/etc/yum.repos.d/` | YUM/DNF repos (RHEL/Amazon Linux) |
| `/etc/dnf/dnf.conf` | DNF settings |

---

## 4. /var — Logs, Caches, State

| Path | Contents |
|------|----------|
| `/var/log/` | All logs |
| `/var/log/messages` | General system log (RHEL) |
| `/var/log/syslog` | General system log (Ubuntu) |
| `/var/log/secure` | Auth/SSH/sudo log (RHEL) |
| `/var/log/auth.log` | Auth/SSH/sudo log (Ubuntu) |
| `/var/log/dmesg` | Kernel boot messages |
| `/var/log/journal/` | Persistent systemd journal |
| `/var/log/nginx/`, `/var/log/httpd/` | Web server logs |
| `/var/log/audit/audit.log` | SELinux/auditd events |
| `/var/log/cron` | Cron execution log (RHEL) |
| `/var/log/dnf.log`, `/var/log/apt/` | Package install history |
| `/var/lib/` | App state: `/var/lib/mysql`, `/var/lib/docker`, `/var/lib/kubelet` |
| `/var/cache/` | Re-creatable cache (`apt`, `dnf`) |
| `/var/spool/` | Queues: cron, mail, print |
| `/var/spool/cron/` | Per-user crontabs |
| `/var/tmp/` | Temp that survives reboot |
| `/var/www/` | Web content (Debian default) |
| `/var/run` → `/run` | Runtime (symlink) |

> 🔥 **`/var` full is the #1 disk alert.** Usual suspects: `/var/log`, `/var/lib/docker`, `/var/log/journal`, `/var/cache`.

```bash
du -xh /var --max-depth=2 2>/dev/null | sort -hr | head -15
journalctl --disk-usage
docker system df
```

---

## 5. /proc — Live Kernel & Process Data

Virtual filesystem — **nothing on disk**. Generated by the kernel on read.

### System-wide

| Path | Shows |
|------|-------|
| `/proc/cpuinfo` | CPU model, cores, flags |
| `/proc/meminfo` | Detailed memory stats |
| `/proc/loadavg` | Load average + running procs |
| `/proc/uptime` | Seconds since boot |
| `/proc/version` | Kernel version string |
| `/proc/cmdline` | Kernel boot parameters |
| `/proc/mounts` | Currently mounted filesystems |
| `/proc/partitions` | Block device partitions |
| `/proc/filesystems` | Supported filesystem types |
| `/proc/swaps` | Active swap areas |
| `/proc/net/tcp` | TCP socket table |
| `/proc/sys/` | Kernel tunables (via `sysctl`) |

### Per-process: `/proc/<PID>/`

| Path | Shows |
|------|-------|
| `cmdline` | Full command that started it |
| `environ` | Process environment variables |
| `cwd` | Symlink to working dir |
| `exe` | Symlink to binary |
| `fd/` | Open file descriptors |
| `status` | State, memory, threads, UID |
| `limits` | Effective ulimits |
| `maps` | Memory mappings |
| `io` | Read/write bytes |
| `oom_score` | OOM-killer likelihood |

```bash
cat /proc/1234/cmdline | tr '\0' ' '        # Full command
tr '\0' '\n' < /proc/1234/environ           # Env vars of process
ls -l /proc/1234/fd | wc -l                 # Count open files (FD leak?)
cat /proc/1234/limits | grep "open files"   # Max open files
ls -l /proc/1234/cwd                        # Where it runs from
grep -E 'VmRSS|Threads' /proc/1234/status   # Memory & threads
cat /proc/sys/fs/file-nr                    # System-wide open files
```

---

## 6. /sys — Hardware & Kernel Tunables

| Path | Purpose |
|------|---------|
| `/sys/block/` | Block devices (disks) |
| `/sys/class/net/` | Network interfaces |
| `/sys/class/net/eth0/statistics/` | Interface counters |
| `/sys/fs/cgroup/` | cgroups (container CPU/mem limits) |
| `/sys/kernel/mm/transparent_hugepage/` | THP setting (DB tuning) |
| `/sys/block/sda/queue/scheduler` | I/O scheduler |

```bash
cat /sys/class/net/eth0/operstate                 # up/down
cat /sys/class/net/eth0/speed                     # Link speed Mbps
cat /sys/block/sda/queue/rotational               # 1=HDD 0=SSD
echo 1 > /sys/class/scsi_host/host0/scan          # Rescan for new disks (cloud)
cat /sys/fs/cgroup/memory.max                     # Container memory limit (cgroup v2)
```

---

## 7. /dev — Device Files

| Device | Meaning |
|--------|---------|
| `/dev/sda`, `/dev/sdb` | SCSI/SATA disks |
| `/dev/sda1` | First partition of sda |
| `/dev/nvme0n1`, `/dev/nvme0n1p1` | NVMe disk & partition (AWS Nitro) |
| `/dev/xvda` | Xen virtual disk (older AWS) |
| `/dev/vda` | KVM virtio disk |
| `/dev/mapper/` | LVM & encrypted volumes |
| `/dev/null` | Discards everything written |
| `/dev/zero` | Infinite zeros |
| `/dev/random`, `/dev/urandom` | Random bytes |
| `/dev/tty`, `/dev/pts/N` | Terminals / SSH sessions |
| `/dev/shm` | Shared memory (tmpfs) |
| `/dev/disk/by-uuid/` | Stable disk names by UUID |

```bash
dd if=/dev/zero of=/tmp/test.img bs=1M count=100   # Create 100MB test file
head -c 32 /dev/urandom | base64                   # Random secret
```

---

## 8. Binaries: /bin, /sbin, /usr, /usr/local, /opt

| Location | Who puts files there | Example |
|----------|----------------------|---------|
| `/usr/bin` | Package manager | `git`, `python3`, `curl` |
| `/usr/sbin` | Package manager (admin) | `sshd`, `useradd`, `nginx` |
| `/usr/local/bin` | **You** (manual installs) | `kubectl`, `terraform`, `helm` |
| `/opt/<vendor>` | Vendor bundles | `/opt/google/chrome`, `/opt/app` |
| `~/.local/bin` | User-only installs | `pip install --user` tools |

> **Rule:** Never manually drop files into `/usr/bin` — package manager owns it. Use `/usr/local/bin` or `/opt`.

---

## 9. Temp & Runtime: /tmp, /var/tmp, /run

| Path | Survives reboot? | Notes |
|------|------------------|-------|
| `/tmp` | ❌ Usually no | Sticky bit `drwxrwxrwt`, often tmpfs, cleaned by `systemd-tmpfiles` (~10 days) |
| `/var/tmp` | ✅ Yes | Cleaned after ~30 days |
| `/run` | ❌ No (tmpfs, RAM) | PID files, sockets, locks |
| `/dev/shm` | ❌ No (RAM) | Shared memory |

---

## 10. Commands to Explore the Hierarchy

```bash
man hier                       # Official FHS description
ls -l /                        # Top-level layout
tree -L 1 /                    # Tree view, 1 level
tree -d -L 2 /etc              # Only dirs, 2 levels
findmnt                        # Mount tree
findmnt /var                   # What's mounted at /var
df -hT                         # Filesystems, type, usage
lsblk -f                       # Disks, partitions, FS, mountpoints
stat -f /                      # Filesystem info for path
namei -l /var/www/html/index.html   # Perms of every path component ← 403 debugging
```

| Option | What it does |
|--------|--------------|
| `tree -L N` | Limit display depth to N |
| `tree -d` | Show directories only, not files |
| `tree -a` | Include hidden files too |
| `df -T` | Show filesystem type column |
| `df -i` | Show inode usage, not blocks |
| `lsblk -f` | Show filesystem, label, UUID |
| `findmnt -t ext4` | List only this filesystem type |
| `namei -l` | Long listing for each path part |

---

## 11. Containers & Cloud Paths

| Path | Purpose |
|------|---------|
| `/var/lib/docker/` | Docker images, containers, volumes |
| `/var/lib/docker/overlay2/` | Container layers (often huge) |
| `/var/lib/containerd/` | containerd data (K8s nodes) |
| `/var/lib/kubelet/` | Kubelet pods & volumes |
| `/etc/kubernetes/` | K8s configs & manifests |
| `/etc/kubernetes/manifests/` | Static pods (control plane) |
| `/var/log/pods/`, `/var/log/containers/` | K8s container logs |
| `/etc/docker/daemon.json` | Docker daemon config |
| `/var/lib/cloud/` | cloud-init state (AWS/Azure/GCP) |
| `/var/log/cloud-init-output.log` | User-data script output ← EC2 bootstrap debugging |
| `/etc/amazon/ssm/` | AWS SSM agent config |

---

## 12. Real-World: Where Do I Look When…

| Situation | Look here |
|-----------|-----------|
| SSH login fails | `/var/log/secure` or `/var/log/auth.log`, `/etc/ssh/sshd_config` |
| DNS not resolving | `/etc/resolv.conf`, `/etc/hosts`, `/etc/nsswitch.conf` |
| Server won't boot after disk change | `/etc/fstab` |
| Service fails to start | `/etc/systemd/system/`, `journalctl -u svc` |
| Disk full alert | `/var/log`, `/var/lib/docker`, `/tmp`, `/home` |
| "Too many open files" | `/proc/PID/limits`, `/etc/security/limits.conf` |
| Kernel update failed | `/boot` space (`df -h /boot`) |
| EC2 user-data didn't run | `/var/log/cloud-init-output.log` |
| Cron didn't run | `/var/log/cron`, `/etc/crontab`, `/var/spool/cron/` |
| Who ran sudo? | `/var/log/secure`, `/var/log/auth.log` |
| Package installed when? | `/var/log/dnf.log`, `/var/log/apt/history.log` |
| OOM kills | `dmesg -T`, `/var/log/messages` |

---

## 13. Interview Questions

1. **`/root` vs `/`?** `/` is filesystem root; `/root` is root user's home.
2. **Is `/proc` on disk?** No — virtual, generated by kernel in memory.
3. **`/tmp` vs `/var/tmp`?** `/tmp` cleared at boot; `/var/tmp` persists.
4. **Where do manually installed binaries go?** `/usr/local/bin` or `/opt`.
5. **Why is `/etc/fstab` dangerous?** A bad line can drop boot into emergency mode — use `nofail` and test with `mount -a`.
6. **How to check open FDs of a process?** `ls /proc/PID/fd | wc -l`
7. **Where are Docker images stored?** `/var/lib/docker/overlay2/`
8. **What is the sticky bit on `/tmp`?** Only a file's owner can delete it.

---

## 14. Cheat Sheet

```text
CONFIG   /etc         passwd shadow group sudoers fstab hosts resolv.conf ssh/sshd_config
LOGS     /var/log     messages|syslog  secure|auth.log  journal/  audit/
STATE    /var/lib     docker kubelet mysql
BINARIES /usr/bin (pkg)  /usr/local/bin (you)  /opt (vendor)
LIVE     /proc/PID/{cmdline,environ,fd,limits,status}   /proc/meminfo
HW       /sys/class/net  /sys/block  /sys/fs/cgroup
DEVICES  /dev/sda /dev/nvme0n1 /dev/mapper /dev/null
RUNTIME  /run  /tmp (sticky)  /var/tmp (persists)
BOOT     /boot  vmlinuz initramfs grub2
EXPLORE  man hier | tree -L 1 / | findmnt | lsblk -f | df -hT | namei -l
```
