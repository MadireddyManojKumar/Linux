# 12 · Service, Network & Process Management

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Run and debug services with systemd, inspect and control processes, and troubleshoot networking end-to-end — the daily core of incident response.

---

## Table of Contents

**Part A — Service Management (systemd)**
1. [systemd Basics](#1-systemd-basics)
2. [systemctl — Control Services](#2-systemctl--control-services)
3. [Writing a Unit File](#3-writing-a-unit-file)
4. [Overrides & Drop-ins](#4-overrides--drop-ins)
5. [journalctl — Service Logs](#5-journalctl--service-logs)
6. [Timers & Cron](#6-timers--cron)
7. [Boot Targets & Boot Analysis](#7-boot-targets--boot-analysis)

**Part B — Process Management**

8. [Viewing Processes — ps, pgrep, pstree](#8-viewing-processes--ps-pgrep-pstree)
9. [Live Monitoring — top, htop](#9-live-monitoring--top-htop)
10. [Signals & Killing Processes](#10-signals--killing-processes)
11. [Jobs, Background & Surviving Logout](#11-jobs-background--surviving-logout)
12. [Priority — nice, renice, ionice](#12-priority--nice-renice-ionice)
13. [Deep Inspection — lsof, strace, /proc](#13-deep-inspection--lsof-strace-proc)
14. [Performance Tools — vmstat, iostat, sar, mpstat, pidstat](#14-performance-tools)
15. [Memory & OOM Killer](#15-memory--oom-killer)
16. [Process States & Zombies](#16-process-states--zombies)

**Part C — Network Management**

17. [Interfaces & IPs — `ip`](#17-interfaces--ips--ip)
18. [Routing](#18-routing)
19. [Ports & Sockets — `ss`, `netstat`, `lsof`](#19-ports--sockets--ss-netstat-lsof)
20. [Connectivity — ping, traceroute, mtr, nc, telnet](#20-connectivity--ping-traceroute-mtr-nc-telnet)
21. [DNS — dig, nslookup, host, resolvectl](#21-dns--dig-nslookup-host-resolvectl)
22. [HTTP — curl & wget](#22-http--curl--wget)
23. [Network Configuration — nmcli, netplan, hostnamectl](#23-network-configuration)
24. [Firewalls — firewalld, ufw, iptables, nftables](#24-firewalls)
25. [Packet Capture & Bandwidth — tcpdump, iftop, ethtool](#25-packet-capture--bandwidth)
26. [TLS / Certificates — openssl](#26-tls--certificates--openssl)
27. [Kernel Network Tuning — sysctl](#27-kernel-network-tuning--sysctl)

**Part D — Putting It Together**

28. [Incident Playbooks](#28-incident-playbooks)
29. [Interview Questions](#29-interview-questions)
30. [Cheat Sheet](#30-cheat-sheet)

---

# Part A — Service Management (systemd)

## 1. systemd Basics

systemd is PID 1 on modern Linux: it starts services, manages dependencies, restarts crashes and collects logs.

| Unit type | Extension | Example |
|-----------|-----------|---------|
| Service | `.service` | `nginx.service` |
| Socket | `.socket` | `sshd.socket` |
| Timer | `.timer` | `logrotate.timer` (cron replacement) |
| Target | `.target` | `multi-user.target` (runlevel) |
| Mount | `.mount` | `data.mount` |
| Path | `.path` | Trigger on file change |

| Location | Purpose | Priority |
|----------|---------|----------|
| `/etc/systemd/system/` | **Your** units & overrides | Highest |
| `/run/systemd/system/` | Runtime units | Middle |
| `/usr/lib/systemd/system/` (`/lib/...`) | Package-provided units — don't edit | Lowest |

---

## 2. systemctl — Control Services

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx              # Stop + start (brief downtime)
sudo systemctl reload nginx               # Re-read config, no downtime ← prefer
sudo systemctl reload-or-restart nginx
sudo systemctl enable nginx               # Start at boot
sudo systemctl enable --now nginx         # Enable + start ← common
sudo systemctl disable --now nginx
sudo systemctl mask nginx                 # Make impossible to start
sudo systemctl unmask nginx
systemctl status nginx                    # State, PID, memory, last logs
systemctl status nginx -l --no-pager      # Full lines
systemctl is-active nginx                 # active / inactive (scripts)
systemctl is-enabled nginx
systemctl is-failed nginx
systemctl --failed                        # All failed units ← incident first check
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-unit-files --state=enabled
systemctl cat nginx                       # Show unit file + overrides
systemctl show nginx -p MainPID,Restart,MemoryCurrent,ActiveEnterTimestamp
systemctl list-dependencies nginx
sudo systemctl daemon-reload              # After editing unit files ← don't forget
sudo systemctl reset-failed               # Clear failed state
sudo systemctl kill -s SIGHUP nginx       # Send signal to service
sudo systemctl edit nginx                 # Create override drop-in
sudo systemctl edit --full nginx          # Full copy to edit
systemctl --user status myapp             # User-level services
```

| Option / Sub-command | What it does |
|----------------------|--------------|
| `start` / `stop` | Start or stop service right now |
| `restart` | Stop then start, brief downtime |
| `reload` | Reload config without stopping service |
| `enable` / `disable` | Start / don't start at boot |
| `--now` | Also start/stop immediately with enable |
| `mask` | Link to /dev/null, block starting |
| `status` | Show state, PID, recent logs |
| `is-active` | Print active/inactive, script friendly |
| `--failed` | List units in failed state |
| `cat` | Print unit file plus drop-ins |
| `show -p` | Print specific unit properties |
| `daemon-reload` | Reload unit files after edits |
| `edit` | Create override drop-in file safely |
| `-l` / `--no-pager` | Full lines / print without pager |
| `-H host` | Run systemctl on remote host |
| `--user` | Manage user's own services |

**`systemctl status` states**

| State | Meaning |
|-------|---------|
| `active (running)` | Running normally |
| `active (exited)` | One-shot finished successfully |
| `inactive (dead)` | Stopped |
| `failed` | Crashed / non-zero exit / start timeout |
| `activating (auto-restart)` | Crash-looping |

---

## 3. Writing a Unit File

`/etc/systemd/system/myapp.service`

```ini
[Unit]
Description=My Application API
Documentation=https://wiki.example.com/myapp
After=network-online.target
Wants=network-online.target
# Requires=postgresql.service

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp/current
EnvironmentFile=-/etc/myapp/myapp.env
Environment=JAVA_OPTS=-Xmx2g
ExecStartPre=/opt/myapp/current/bin/check-config
ExecStart=/opt/myapp/current/bin/server --port 8080
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
StartLimitIntervalSec=300
StartLimitBurst=5
TimeoutStartSec=60
TimeoutStopSec=30
LimitNOFILE=65535
MemoryMax=2G
CPUQuota=200%
StandardOutput=journal
StandardError=journal
# Hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemd-analyze verify /etc/systemd/system/myapp.service   # Lint unit file
```

| Directive | What it does |
|-----------|--------------|
| `After=` | Start after these units (ordering only) |
| `Wants=` / `Requires=` | Soft / hard dependency on unit |
| `Type=simple` | Main process is ExecStart itself |
| `Type=forking` | Process forks, parent exits (legacy daemons) |
| `Type=oneshot` | Runs once then exits (scripts) |
| `Type=notify` | Service signals readiness to systemd |
| `User=` / `Group=` | Run service as this user |
| `EnvironmentFile=` | Load env vars from file (`-` = optional) |
| `ExecStart=` | Command that starts the service |
| `ExecStartPre=` | Command run before main start |
| `Restart=on-failure` | Restart if exits with error |
| `Restart=always` | Always restart, even clean exit |
| `RestartSec=` | Wait time before restarting service |
| `StartLimitBurst=` | Max restarts within interval window |
| `LimitNOFILE=` | Max open file descriptors for service |
| `MemoryMax=` | Hard memory limit via cgroup |
| `CPUQuota=` | Max CPU share (100% = 1 core) |
| `WantedBy=multi-user.target` | Start at normal multi-user boot |

---

## 4. Overrides & Drop-ins

Never edit `/usr/lib/systemd/system/*` — package updates overwrite it.

```bash
sudo systemctl edit nginx
# Creates /etc/systemd/system/nginx.service.d/override.conf
```
```ini
[Service]
LimitNOFILE=100000
Restart=always
# To replace ExecStart you must clear it first:
ExecStart=
ExecStart=/usr/sbin/nginx -g 'daemon off;'
```
```bash
sudo systemctl daemon-reload && sudo systemctl restart nginx
systemd-delta                               # Show all overridden units
```

---

## 5. journalctl — Service Logs

```bash
journalctl -u myapp                         # All logs for service
journalctl -u myapp -f                      # Follow live
journalctl -u myapp -n 200 --no-pager       # Last 200 lines
journalctl -u myapp --since "30 min ago"
journalctl -u myapp --since today -p err    # Errors today
journalctl -u myapp -b                      # Since last boot
journalctl -b -1 -p err                     # Errors from PREVIOUS boot (post-crash)
journalctl -k                               # Kernel ring buffer
journalctl -u nginx -u myapp --since "10:00" --until "10:30"
journalctl -o json-pretty -u myapp | head
journalctl --list-boots
journalctl --disk-usage
sudo journalctl --vacuum-size=1G            # Shrink journal
sudo journalctl --vacuum-time=7d
```

| Option | What it does |
|--------|--------------|
| `-u UNIT` | Logs for this unit only |
| `-f` | Follow new messages in real time |
| `-n N` | Show last N lines |
| `-p LEVEL` | Minimum priority: emerg..debug |
| `-b [-N]` | Current or N-th previous boot |
| `-k` | Kernel messages only |
| `--since / --until` | Time range for logs |
| `-o FORMAT` | Output: short, json, cat, verbose |
| `-r` | Newest entries first |
| `-x` | Add explanations to messages |
| `--vacuum-size=` | Delete old logs above size |
| `--vacuum-time=` | Delete logs older than time |

Persistent journal: set `Storage=persistent` in `/etc/systemd/journald.conf` (logs in `/var/log/journal`).

---

## 6. Timers & Cron

### systemd timer

```ini
# /etc/systemd/system/backup.service
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

# /etc/systemd/system/backup.timer
[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true
RandomizedDelaySec=10m

[Install]
WantedBy=timers.target
```
```bash
sudo systemctl enable --now backup.timer
systemctl list-timers --all
systemd-analyze calendar "Mon..Fri 09:00"   # Validate schedule
```

### cron

```bash
crontab -e            # Edit your crontab
crontab -l            # List
crontab -r            # Remove all ⚠️
sudo crontab -u app -l
```

```
┌──── minute (0-59)
│ ┌──── hour (0-23)
│ │ ┌──── day of month (1-31)
│ │ │ ┌──── month (1-12)
│ │ │ │ ┌──── day of week (0-7, Sun=0/7)
* * * * *  command
```

| Example | Runs |
|---------|------|
| `*/5 * * * *` | Every 5 minutes |
| `0 2 * * *` | Daily at 02:00 |
| `30 1 * * 0` | Sundays 01:30 |
| `0 9-17 * * 1-5` | Hourly, 9–5, weekdays |
| `@reboot` | At boot |

```cron
*/5 * * * * /opt/scripts/health.sh >> /var/log/health.log 2>&1
```

> Cron has a minimal `PATH` and no profile — use **full paths**, redirect output, and check `/var/log/cron` or `journalctl -u cron`/`crond`.

**One-off jobs:** `echo "systemctl restart app" | at 02:00` · `atq` · `atrm N`

---

## 7. Boot Targets & Boot Analysis

| Target | Old runlevel | Meaning |
|--------|--------------|---------|
| `poweroff.target` | 0 | Shutdown |
| `rescue.target` | 1 | Single user |
| `multi-user.target` | 3 | Servers (no GUI) |
| `graphical.target` | 5 | GUI |
| `reboot.target` | 6 | Reboot |
| `emergency.target` | — | Minimal shell (e.g., bad fstab) |

```bash
systemctl get-default
sudo systemctl set-default multi-user.target
sudo systemctl isolate rescue.target
systemd-analyze                   # Total boot time
systemd-analyze blame | head      # Slowest units
systemd-analyze critical-chain
sudo systemctl reboot / poweroff
sudo shutdown -r +10 "Patching reboot in 10 min"
sudo shutdown -c                  # Cancel
```

---

# Part B — Process Management

## 8. Viewing Processes — ps, pgrep, pstree

```bash
ps aux                                    # All processes, BSD style ← most used
ps -ef                                    # All processes, full format (PPID)
ps aux --sort=-%cpu | head                # Top CPU
ps aux --sort=-%mem | head                # Top memory
ps -eo pid,ppid,user,%cpu,%mem,etime,stat,cmd --sort=-%cpu | head
ps -p 1234 -o pid,etime,lstart,cmd        # Uptime of a process
ps -u nginx                               # Processes of a user
ps -fC java                               # By command name
ps -eLf | wc -l                           # Total threads
ps -T -p 1234                             # Threads of a process
ps axjf                                   # Process tree
pstree -p                                 # Tree with PIDs
pstree -ap 1234
pgrep -a nginx                            # PIDs + command lines
pgrep -u app -f "java.*order-service"     # Match full cmdline
pidof nginx
```

| Option | What it does |
|--------|--------------|
| `a` | Processes of all users |
| `u` | User-oriented format with CPU/MEM |
| `x` | Include processes without a terminal |
| `-e` | Select every process on system |
| `-f` | Full format, shows PPID, cmd |
| `-o` | Choose custom output columns |
| `--sort=-%cpu` | Sort by CPU, descending |
| `-p PID` | Only this process ID |
| `-u USER` | Processes owned by user |
| `-C NAME` | Processes by command name |
| `-L` / `-T` | Show threads |
| `pgrep -f` | Match against full command line |
| `pgrep -a` | Print PID and full command |
| `pgrep -u` | Only processes of this user |

**`ps aux` columns:** `USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND` — **RSS** = real RAM used (KB), **VSZ** = virtual size.

---

## 9. Live Monitoring — top, htop

```bash
top
top -o %MEM                   # Sort by memory
top -p 1234,5678              # Only these PIDs
top -u nginx
top -b -n 1 | head -20        # Batch mode snapshot (scripts/tickets)
top -H -p 1234                # Threads of one process (hot Java thread)
htop                          # Interactive, colored, tree view (F5)
btop / glances / atop         # Alternatives (atop records history)
```

| Option | What it does |
|--------|--------------|
| `-o FIELD` | Sort by this field |
| `-p PID` | Monitor only these processes |
| `-u USER` | Show only this user's processes |
| `-b` | Batch mode for logging output |
| `-n N` | Exit after N refreshes |
| `-d SEC` | Refresh delay in seconds |
| `-H` | Show individual threads |

**Keys in top:** `P` sort CPU · `M` sort mem · `1` per-CPU · `c` full cmd · `k` kill · `H` threads · `q` quit

**top header decoded**

```
load average: 4.10, 3.50, 2.00      ← compare to nproc
%Cpu(s): 25.0 us, 5.0 sy, 0.0 ni, 60.0 id, 10.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

| Field | Meaning | High means |
|-------|---------|------------|
| `us` | User CPU | App is busy |
| `sy` | Kernel CPU | Syscalls, context switching |
| `wa` | I/O wait | **Disk bottleneck** |
| `st` | Steal | **Noisy neighbor / VM CPU starved** |
| `id` | Idle | Spare capacity |

---

## 10. Signals & Killing Processes

```bash
kill 1234                     # SIGTERM (15): graceful stop
kill -9 1234                  # SIGKILL: force, can't be caught ← last resort
kill -HUP 1234                # Reload config (nginx, sshd)
kill -0 1234 && echo alive    # Check if process exists
kill -l                       # List signals
pkill nginx                   # By name
pkill -f "python app.py"      # By full command
pkill -u baduser              # All processes of user
killall -9 java               # All by exact name
timeout 30s ./script.sh       # Kill if runs >30s
timeout -s KILL 10 cmd
```

| Signal | No. | What it does |
|--------|-----|--------------|
| `SIGHUP` | 1 | Reload config / terminal closed |
| `SIGINT` | 2 | Interrupt, same as Ctrl+C |
| `SIGQUIT` | 3 | Quit + core dump (Java: thread dump) |
| `SIGKILL` | 9 | Force kill, cannot be caught |
| `SIGTERM` | 15 | Polite stop request (default) |
| `SIGSTOP` | 19 | Pause process, cannot be caught |
| `SIGCONT` | 18 | Resume a paused process |
| `SIGUSR1/2` | 10/12 | App-defined (log reopen, etc.) |

> Always try `SIGTERM` first, wait, then `SIGKILL`. `-9` skips cleanup (temp files, locks, connections).

---

## 11. Jobs, Background & Surviving Logout

```bash
./long.sh &                  # Run in background
jobs -l                      # List jobs with PIDs
fg %1                        # Bring job 1 to foreground
bg %1                        # Resume stopped job in background
Ctrl+Z                       # Suspend foreground job
disown -h %1                 # Detach job from shell
nohup ./long.sh > out.log 2>&1 &     # Survive logout
setsid ./long.sh             # New session, fully detached
```

**Terminal multiplexers** — keep sessions alive through SSH drops:
```bash
tmux new -s incident         # New session
Ctrl+b d                     # Detach
tmux ls ; tmux attach -t incident
screen -S work ; Ctrl+a d ; screen -r work
```

---

## 12. Priority — nice, renice, ionice

Nice range: **-20 (highest priority)** to **19 (lowest)**. Only root can go negative.

```bash
nice -n 10 ./backup.sh              # Start with lower priority
sudo renice -n -5 -p 1234           # Raise priority of running PID
sudo renice -n 15 -u batchuser      # All processes of user
ionice -c3 -p 1234                  # Idle I/O class (only when disk free)
ionice -c2 -n7 tar -czf ...         # Best-effort, lowest
nice -n 19 ionice -c3 rsync ...     # Friendliest background copy
taskset -cp 0,1 1234                # Pin process to CPUs 0 and 1
```

| Option | What it does |
|--------|--------------|
| `nice -n N` | Start command with niceness N |
| `renice -n N -p` | Change niceness of running PID |
| `renice -u USER` | Apply to all user's processes |
| `ionice -c1/2/3` | Realtime / best-effort / idle I/O |
| `ionice -n 0-7` | Priority within class, 7 lowest |
| `taskset -cp` | Set CPU affinity for PID |

---

## 13. Deep Inspection — lsof, strace, /proc

### lsof — list open files

```bash
sudo lsof -p 1234                        # Everything a process has open
sudo lsof -i :8080                       # Who uses port 8080
sudo lsof -i TCP -s TCP:LISTEN           # Listening TCP
sudo lsof -i @10.0.1.5                   # Connections to host
sudo lsof -u nginx                       # Files by user
sudo lsof /var/log/app.log               # Who holds this file
sudo lsof +D /data                       # Everything open under dir (umount busy)
sudo lsof +L1                            # Deleted-but-open files ← disk not freed
sudo lsof -nP -i                         # All network, no DNS/port names (fast)
sudo lsof -p 1234 | wc -l                # FD count
```

| Option | What it does |
|--------|--------------|
| `-p PID` | Files opened by this process |
| `-i :PORT` | Network files using this port |
| `-i TCP/UDP` | Only TCP or UDP sockets |
| `-u USER` | Files opened by this user |
| `+D DIR` | Recursively, files open under directory |
| `+L1` | Deleted files still held open |
| `-n` / `-P` | No hostname / no port name lookup |
| `-t` | Output PIDs only (for kill) |

### fuser

```bash
sudo fuser -v 8080/tcp                   # Who uses port
sudo fuser -vm /data                     # Who uses mount
sudo fuser -km /data                     # Kill them ⚠️
```

### strace — trace system calls

```bash
sudo strace -p 1234                      # Attach to running process
sudo strace -f -p 1234                   # Follow child threads/forks
sudo strace -e trace=open,openat,read ./app   # Only file calls
sudo strace -e trace=network -p 1234     # Network calls
sudo strace -c -p 1234                   # Syscall summary (Ctrl+C to stop)
sudo strace -tt -T -o /tmp/trace.txt ./app    # Timestamps + time in call
strace -f -e trace=file ./app 2>&1 | grep ENOENT   # Missing files/config
```

| Option | What it does |
|--------|--------------|
| `-p PID` | Attach to running process ID |
| `-f` | Also trace forked children/threads |
| `-e trace=X` | Only trace these syscall categories |
| `-c` | Count calls, show summary table |
| `-tt` | Microsecond timestamps on each line |
| `-T` | Time spent inside each syscall |
| `-o FILE` | Write trace output to file |
| `-s N` | Print N chars of strings |

> Also: `ltrace` (library calls), `perf top` (CPU hotspots), `gdb -p` (debugger), `jstack`/`jcmd` (Java), `py-spy` (Python).

---

## 14. Performance Tools

**USE method:** for each resource (CPU, memory, disk, network), check **U**tilization, **S**aturation, **E**rrors.

```bash
uptime                         # Load averages
vmstat 1 5                     # CPU, memory, swap, I/O every 1s ×5
mpstat -P ALL 1                # Per-CPU usage (sysstat)
pidstat 1                      # Per-process CPU
pidstat -r 1                   # Per-process memory
pidstat -d 1                   # Per-process disk I/O
iostat -xz 1                   # Disk utilization & latency
sar -u 1 5                     # CPU (sar also has history)
sar -r / -n DEV / -d           # Memory / network / disk
sar -u -f /var/log/sa/sa26     # CPU history for the 26th ← "what happened at 3am?"
free -h                        # Memory
dmesg -T | tail                # Kernel errors
```

**`vmstat` key columns**

| Col | Meaning | Watch for |
|-----|---------|-----------|
| `r` | Runnable procs | > CPU count = CPU saturation |
| `b` | Blocked on I/O | > 0 sustained = disk issue |
| `si`/`so` | Swap in/out | > 0 = memory pressure |
| `wa` | I/O wait % | High = slow disk |
| `st` | Steal % | High = VM starved |

| Option | What it does |
|--------|--------------|
| `vmstat 1 5` | Every 1 second, 5 samples |
| `vmstat -s` | Memory statistics summary table |
| `mpstat -P ALL` | Stats for every CPU core |
| `pidstat -r/-d/-u` | Memory / disk / CPU per process |
| `sar -f FILE` | Read historical data from file |

**Netflix "60-second" checklist:** `uptime` → `dmesg -T | tail` → `vmstat 1` → `mpstat -P ALL 1` → `pidstat 1` → `iostat -xz 1` → `free -m` → `sar -n DEV 1` → `sar -n TCP,ETCP 1` → `top`

---

## 15. Memory & OOM Killer

```bash
free -h
#               total   used   free   shared  buff/cache   available
# Mem:           15Gi   6Gi    1Gi    0.3Gi   8Gi          8.5Gi   ← "available" is what matters
cat /proc/meminfo | grep -E 'MemAvailable|SwapTotal|SwapFree|Dirty'
ps aux --sort=-rss | head
smem -rs rss                          # Proportional memory (PSS)
pmap -x 1234 | tail -1                # Memory map total of process
dmesg -T | grep -iE 'killed process|out of memory'   # OOM events
journalctl -k | grep -i oom
cat /proc/1234/oom_score
echo -1000 | sudo tee /proc/1234/oom_score_adj       # Protect process from OOM
sync; echo 3 | sudo tee /proc/sys/vm/drop_caches     # Drop caches (testing only)
```

> "Free" memory being low is normal — Linux uses spare RAM for cache. Look at **`available`**, swap activity (`si/so`), and OOM logs.

---

## 16. Process States & Zombies

| STAT | Meaning |
|------|---------|
| `R` | Running / runnable |
| `S` | Sleeping (waiting for event) |
| `D` | Uninterruptible sleep (usually disk/NFS I/O) — can't be killed |
| `Z` | Zombie (exited, parent hasn't reaped) |
| `T` | Stopped |
| `<` / `N` | High / low priority |
| `s` / `l` / `+` | Session leader / multithreaded / foreground |

```bash
ps aux | awk '$8 ~ /Z/'                        # Zombies
ps -o ppid= -p <zombie_pid>                    # Its parent → fix/restart parent
ps -eo stat,pid,cmd | awk '$1 ~ /D/'           # Stuck in D state → check storage/NFS
```

> Zombies use no resources except a PID slot — kill/restart the **parent**. Many `D` processes = storage problem.

---

# Part C — Network Management

## 17. Interfaces & IPs — `ip`

`ip` (iproute2) replaces deprecated `ifconfig`, `route`, `arp`.

```bash
ip a                                  # All interfaces + IPs (ip addr show)
ip -br a                              # Brief one-line view ← quick
ip -4 a show eth0                     # IPv4 only for eth0
ip link                               # Link layer (MAC, state, MTU)
ip -s link show eth0                  # Packet/error/drop counters
sudo ip link set eth0 up / down
sudo ip link set eth0 mtu 9001        # Jumbo frames (AWS)
sudo ip addr add 10.0.1.50/24 dev eth0    # Temporary IP
sudo ip addr del 10.0.1.50/24 dev eth0
ip neigh                              # ARP table
sudo ip neigh flush dev eth0
ip -c a                               # Colorized
```

| Option / Object | What it does |
|-----------------|--------------|
| `a` / `addr` | Show or manage IP addresses |
| `link` | Show or manage network interfaces |
| `route` / `r` | Show or manage routing table |
| `neigh` | Show ARP / neighbor cache |
| `-br` | Brief, one line per interface |
| `-4` / `-6` | Only IPv4 / IPv6 |
| `-s` | Include statistics: packets, errors, drops |
| `-c` | Colorize output for readability |

> Changes via `ip` are **temporary** — lost at reboot. Persist with nmcli / netplan (§23).

---

## 18. Routing

```bash
ip route                              # Routing table (ip r)
ip route get 8.8.8.8                  # Which route/interface/source IP is used ← great
sudo ip route add 10.20.0.0/16 via 10.0.1.1 dev eth0
sudo ip route del 10.20.0.0/16
sudo ip route add default via 10.0.1.1
ip rule                               # Policy routing rules
```

| Field | Meaning |
|-------|---------|
| `default via 10.0.1.1` | Default gateway |
| `dev eth0` | Outgoing interface |
| `src 10.0.1.15` | Source IP used |
| `proto dhcp` / `kernel` / `static` | Route origin |

---

## 19. Ports & Sockets — `ss`, `netstat`, `lsof`

```bash
sudo ss -tulnp                        # TCP+UDP listening + process ← memorize
sudo ss -tlnp | grep :443
ss -tan                               # All TCP connections
ss -tan state established | wc -l     # Count established
ss -tan state time-wait | wc -l       # TIME_WAIT buildup
ss -s                                 # Summary stats
ss -tn dst 10.0.3.50                  # Connections to a host
ss -tn sport = :8080                  # Connections from local port 8080
ss -tnp | awk 'NR>1{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head   # Top remote IPs
ss -ti                                # TCP internals (rtt, cwnd, retrans)
ss -x                                 # Unix sockets
netstat -tulnp                        # Legacy equivalent (net-tools)
netstat -s | grep -i retrans          # Retransmission stats
```

| Option | What it does |
|--------|--------------|
| `-t` | Show TCP sockets |
| `-u` | Show UDP sockets |
| `-l` | Only listening sockets |
| `-n` | Numeric, no DNS/service name lookup |
| `-p` | Show owning process and PID |
| `-a` | Show all sockets, listening and not |
| `-s` | Print summary socket statistics |
| `-i` | Show internal TCP info (rtt, retrans) |
| `-x` | Show Unix domain sockets |
| `state X` | Filter by TCP state |

**TCP states to know**

| State | Meaning |
|-------|---------|
| `LISTEN` | Waiting for connections |
| `ESTABLISHED` | Active connection |
| `TIME_WAIT` | Closed by us, waiting (normal; huge numbers = connection churn) |
| `CLOSE_WAIT` | Remote closed, **our app didn't** → app bug / connection leak |
| `SYN_SENT` | Trying to connect, no reply → firewall/remote down |
| `SYN_RECV` | Half-open; many = SYN flood or backlog full |

> `0.0.0.0:8080` = listening on all interfaces. `127.0.0.1:8080` = **localhost only** — a common "works locally, not remotely" cause.

---

## 20. Connectivity — ping, traceroute, mtr, nc, telnet

```bash
ping -c 4 10.0.1.1                    # 4 packets
ping -c 4 -W 2 host                   # 2s timeout
ping -s 8972 -M do host               # MTU test (no fragmentation)
traceroute host                       # Path hops (UDP)
traceroute -T -p 443 host             # TCP traceroute through firewalls
tracepath host                        # No root needed, finds MTU
mtr -rw -c 100 host                   # Report: loss & latency per hop ← best
nc -zv db.internal 5432               # Is TCP port open? ← most used
nc -zv -w 3 host 20-25                # Port range, 3s timeout
nc -zvu host 53                       # UDP
nc -l 9000                            # Listen (test server)
echo > /dev/tcp/host/443 && echo open # Bash built-in port check (no tools)
telnet host 25                        # Legacy port test
arping -I eth0 10.0.1.1               # Layer 2 reachability
```

| Option | What it does |
|--------|--------------|
| `ping -c N` | Send N packets then stop |
| `ping -W N` | Wait N seconds for reply |
| `ping -i N` | Interval between packets in seconds |
| `ping -s SIZE` | Payload size for MTU tests |
| `traceroute -T` | Use TCP SYN instead of UDP |
| `traceroute -n` | Don't resolve hop hostnames |
| `mtr -r` | Report mode, print and exit |
| `mtr -c N` | Send N probes per hop |
| `nc -z` | Scan only, send no data |
| `nc -v` | Verbose, show success/failure |
| `nc -w N` | Timeout after N seconds |
| `nc -u` | Use UDP instead of TCP |
| `nc -l` | Listen mode, act as server |

> Cloud VMs often block ICMP — **ping failing ≠ host down**. Test the actual port with `nc -zv`.

---

## 21. DNS — dig, nslookup, host, resolvectl

```bash
dig example.com                       # Full answer
dig +short example.com                # Just the IP(s)
dig example.com A / AAAA / MX / TXT / NS / CNAME / SOA
dig @8.8.8.8 example.com              # Query a specific resolver
dig +trace example.com                # Full delegation path from root
dig -x 10.0.1.15                      # Reverse lookup (PTR)
dig +noall +answer example.com        # Clean answer section with TTL
nslookup example.com 10.0.0.2
host example.com
getent hosts example.com              # Resolve like apps do (uses /etc/hosts + nsswitch) ← important
resolvectl status                     # systemd-resolved: servers per interface
resolvectl query example.com
sudo resolvectl flush-caches
cat /etc/resolv.conf ; cat /etc/hosts
```

| Option | What it does |
|--------|--------------|
| `+short` | Print only the answer values |
| `@SERVER` | Ask this DNS server directly |
| `+trace` | Trace delegation from root servers |
| `-x IP` | Reverse lookup IP to name |
| `+noall +answer` | Show only the answer section |
| `TYPE` | Record type: A, MX, TXT, etc. |

> `dig` bypasses `/etc/hosts`; applications don't. If `dig` works but the app fails, check `getent hosts`, `/etc/hosts`, and `/etc/nsswitch.conf`.

---

## 22. HTTP — curl & wget

```bash
curl https://api.example.com/health
curl -I https://example.com                         # Headers only
curl -sS -o /dev/null -w "%{http_code}\n" URL       # Status code only (scripts)
curl -v https://example.com                         # Verbose: DNS, TLS, headers
curl -L URL                                         # Follow redirects
curl -k https://self-signed                         # Skip TLS verify (testing only)
curl -X POST -H "Content-Type: application/json" -d '{"k":"v"}' URL
curl -u user:pass URL                               # Basic auth
curl -H "Authorization: Bearer $TOKEN" URL
curl --resolve app.example.com:443:10.0.1.15 https://app.example.com   # Test a specific backend
curl -H "Host: app.example.com" http://10.0.1.15/   # Virtual-host test
curl --connect-timeout 5 -m 10 URL                  # Timeouts
curl --retry 3 --retry-delay 2 URL
curl -x http://proxy:8080 URL                       # Via proxy
curl -O https://host/file.tgz                       # Save with remote name
curl -w "dns:%{time_namelookup} connect:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n" -o /dev/null -s URL   # Latency breakdown ← gold
wget https://host/file.tgz
wget -c URL                                         # Resume download
wget -q -O - URL                                    # Print to stdout
```

| Option | What it does |
|--------|--------------|
| `-I` | Fetch response headers only (HEAD) |
| `-v` | Verbose: connection, TLS, headers |
| `-s` / `-S` | Silent / but still show errors |
| `-o FILE` | Write body to file |
| `-O` | Save using remote file name |
| `-L` | Follow HTTP redirects |
| `-k` | Ignore TLS certificate errors |
| `-X METHOD` | HTTP method: POST, PUT, DELETE |
| `-H` | Add a request header |
| `-d` | Send request body data |
| `-u` | Basic auth user:password |
| `-w FORMAT` | Print timing/status after transfer |
| `--resolve` | Force hostname to specific IP |
| `--connect-timeout` | Max seconds to establish connection |
| `-m` | Max total seconds for request |
| `-x` | Use this proxy server |
| `wget -c` | Continue partially downloaded file |

---

## 23. Network Configuration

### RHEL 8/9 — NetworkManager (`nmcli`)

```bash
nmcli device status
nmcli con show
nmcli con show "System eth0"
sudo nmcli con mod "System eth0" ipv4.addresses 10.0.1.50/24 ipv4.gateway 10.0.1.1 \
     ipv4.dns "10.0.0.2 8.8.8.8" ipv4.method manual
sudo nmcli con up "System eth0"
sudo nmcli con mod eth0 +ipv4.addresses 10.0.1.51/24     # Add secondary IP
nmtui                                                    # Text UI
```

| Option | What it does |
|--------|--------------|
| `device status` | Show all devices and states |
| `con show` | List connection profiles |
| `con mod` | Modify a connection profile |
| `con up / down` | Activate or deactivate connection |
| `ipv4.method manual` | Use static IP, not DHCP |

### Ubuntu — Netplan

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [10.0.1.50/24]
      routes:
        - to: default
          via: 10.0.1.1
      nameservers:
        addresses: [10.0.0.2, 8.8.8.8]
```
```bash
sudo netplan try          # Apply with auto-rollback if you lose connection ← safe
sudo netplan apply
```

### Hostname

```bash
sudo hostnamectl set-hostname web01.prod.example.com
```

---

## 24. Firewalls

### firewalld (RHEL)

```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --remove-port=8080/tcp
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" port port="5432" protocol="tcp" accept'
sudo firewall-cmd --reload
sudo firewall-cmd --add-port=9000/tcp --timeout=1h      # Temporary rule
```

| Option | What it does |
|--------|--------------|
| `--list-all` | Show all rules in zone |
| `--permanent` | Save rule across reloads/reboots |
| `--add-service=` | Allow predefined service by name |
| `--add-port=` | Open port/protocol like 8080/tcp |
| `--zone=` | Apply rule to this zone |
| `--reload` | Apply permanent rules now |
| `--timeout=` | Rule auto-expires after duration |

### ufw (Ubuntu)

```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw allow from 10.0.0.0/8 to any port 5432
sudo ufw deny 23
sudo ufw delete allow 8080
sudo ufw enable      # ⚠️ allow SSH FIRST
```

### iptables / nftables

```bash
sudo iptables -L -n -v --line-numbers          # List rules with counters
sudo iptables -t nat -L -n -v                  # NAT table (Docker/K8s)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -I INPUT 1 -s 1.2.3.4 -j DROP    # Block IP at top
sudo iptables -D INPUT 3                       # Delete rule 3
sudo iptables-save > /root/iptables.bak
sudo iptables-restore < /root/iptables.bak
sudo nft list ruleset                          # nftables (modern backend)
```

| Option | What it does |
|--------|--------------|
| `-L` | List rules in chain |
| `-n` | Numeric output, no DNS lookups |
| `-v` | Show packet and byte counters |
| `-t nat` | Work on the NAT table |
| `-A` | Append rule to end of chain |
| `-I CHAIN N` | Insert rule at position N |
| `-D` | Delete rule by spec or number |
| `-p / --dport` | Protocol / destination port match |
| `-s` | Match source IP or network |
| `-j` | Target action: ACCEPT, DROP, REJECT |

> Remember cloud layers too: **AWS Security Groups / NACLs**, Azure NSGs, GCP firewall rules, K8s NetworkPolicies.

---

## 25. Packet Capture & Bandwidth

```bash
sudo tcpdump -i eth0                                   # Everything
sudo tcpdump -i any port 443 -nn                       # Port 443, no name resolution
sudo tcpdump -i eth0 host 10.0.3.50 and port 5432
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'    # SYN packets only
sudo tcpdump -i eth0 -c 100 -w /tmp/cap.pcap           # Save for Wireshark
sudo tcpdump -r /tmp/cap.pcap -nn                      # Read capture
sudo tcpdump -i eth0 -A port 80                        # Print payload ASCII
sudo tcpdump -i any -nn udp port 53                    # DNS traffic
```

| Option | What it does |
|--------|--------------|
| `-i IFACE` | Capture on this interface (`any` = all) |
| `-n` / `-nn` | No DNS / also no port names |
| `-c N` | Stop after N packets |
| `-w FILE` | Write raw packets to pcap file |
| `-r FILE` | Read packets from pcap file |
| `-A` | Print packet payload as ASCII |
| `-X` | Print payload in hex and ASCII |
| `-s 0` | Capture full packet length |
| `-v` | More verbose protocol details |

```bash
iftop -i eth0                # Live bandwidth per connection
nload eth0                   # In/out throughput
nethogs                      # Bandwidth per process
iperf3 -s   /   iperf3 -c server -t 10   # Throughput test between hosts
ethtool eth0                 # Speed, duplex, link detected
ethtool -S eth0 | grep -iE 'drop|err'   # NIC-level errors
nmap -Pn -p 22,80,443 host   # Port scan (only with authorization)
```

---

## 26. TLS / Certificates — openssl

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null   # TLS handshake
echo | openssl s_client -connect host:443 -servername host 2>/dev/null | openssl x509 -noout -dates -subject -issuer   # Expiry ← very common
openssl x509 -in cert.pem -noout -text              # Inspect cert file
openssl x509 -in cert.pem -noout -enddate
openssl x509 -in cert.pem -noout -checkend 2592000  # Expires in 30 days? (exit 1 = yes)
openssl s_client -connect host:443 -showcerts       # Full chain
openssl verify -CAfile chain.pem cert.pem           # Validate chain
openssl req -new -newkey rsa:2048 -nodes -keyout app.key -out app.csr   # CSR
openssl rsa -noout -modulus -in app.key | md5sum    # Key/cert match check
openssl x509 -noout -modulus -in app.crt | md5sum   #   (hashes must match)
```

| Option | What it does |
|--------|--------------|
| `s_client -connect` | Open TLS connection to host:port |
| `-servername` | Send SNI hostname (required for most) |
| `-showcerts` | Print entire certificate chain |
| `x509 -noout` | Don't print encoded certificate |
| `-dates` | Show notBefore and notAfter dates |
| `-text` | Human-readable full certificate details |
| `-checkend SEC` | Check expiry within N seconds |

---

## 27. Kernel Network Tuning — sysctl

```bash
sysctl -a | grep net.ipv4.tcp
sysctl net.core.somaxconn
sudo sysctl -w net.core.somaxconn=4096               # Temporary
echo "net.core.somaxconn=4096" | sudo tee /etc/sysctl.d/99-tuning.conf
sudo sysctl --system                                 # Reload all sysctl files
```

| Parameter | Purpose |
|-----------|---------|
| `net.core.somaxconn` | Listen backlog queue size |
| `net.ipv4.ip_local_port_range` | Ephemeral ports for outbound connections |
| `net.ipv4.tcp_tw_reuse` | Reuse TIME_WAIT sockets for outbound |
| `net.ipv4.tcp_fin_timeout` | FIN-WAIT-2 timeout |
| `net.ipv4.ip_forward` | Enable routing (Docker/K8s need 1) |
| `net.core.netdev_max_backlog` | NIC receive queue |
| `fs.file-max` | System-wide max open files |
| `vm.swappiness` | Swap tendency (DB servers: 1–10) |

| Option | What it does |
|--------|--------------|
| `-a` | Show all kernel parameters |
| `-w` | Write value, temporary until reboot |
| `-p FILE` | Load settings from a file |
| `--system` | Load all sysctl config directories |

---

# Part D — Putting It Together

## 28. Incident Playbooks

### 🔴 Service is down

```bash
systemctl status myapp -l --no-pager
journalctl -u myapp -n 100 --no-pager
journalctl -u myapp -b -p err
sudo ss -tlnp | grep 8080          # Is it listening? On 0.0.0.0 or 127.0.0.1?
curl -sv localhost:8080/health
df -h ; free -h                    # Disk full? OOM?
dmesg -T | grep -i -E 'oom|killed'
sudo systemctl restart myapp && systemctl is-active myapp
```

### 🔴 High CPU

```bash
uptime ; nproc
top -o %CPU                        # Which process? us vs sy vs wa vs st?
ps -eo pid,ppid,%cpu,etime,cmd --sort=-%cpu | head
top -H -p <PID>                    # Hot thread
sudo perf top -p <PID>             # Hot function
# Java: jstack <PID> — convert hot thread id to hex, find in dump
```

### 🔴 High memory / OOM

```bash
free -h
ps aux --sort=-rss | head
dmesg -T | grep -i "killed process"
cat /sys/fs/cgroup/memory.max      # Container limit?
```

### 🔴 "Can't connect to X"

```
Layer by layer:
1. DNS          dig +short X ; getent hosts X
2. Route        ip route get <IP>
3. Reachability ping / mtr (ICMP may be blocked)
4. Port         nc -zv X PORT
5. Listening    (on X) ss -tlnp | grep PORT   — bound to 0.0.0.0?
6. Firewall     firewall-cmd --list-all / iptables -L -n / Security Groups / NACLs
7. App / TLS    curl -v ; openssl s_client
8. Capture      tcpdump -i any host X and port PORT -nn
```

### 🔴 Slow application

```bash
curl -w "dns:%{time_namelookup} conn:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n" -o /dev/null -s URL
vmstat 1 ; iostat -xz 1            # CPU wait? Disk latency?
ss -tan state close-wait | wc -l   # Connection leak?
ss -ti dst <backend>               # Retransmits / RTT
sar -n TCP,ETCP 1                  # Retransmission rate
```

### 🔴 Port already in use

```bash
sudo ss -tlnp | grep :8080
sudo lsof -i :8080
sudo fuser -k 8080/tcp             # Kill holder (careful)
```

---

## 29. Interview Questions

1. **`restart` vs `reload`?** Restart stops/starts (downtime); reload re-reads config in place.
2. **`enable` vs `start`?** Enable = at boot; start = now. `enable --now` does both.
3. **After editing a unit file?** `systemctl daemon-reload`.
4. **SIGTERM vs SIGKILL?** TERM is graceful & catchable; KILL is immediate, no cleanup.
5. **What is a zombie process? Fix?** Exited child not reaped; restart/fix parent.
6. **Process in `D` state?** Uninterruptible I/O wait — investigate disk/NFS.
7. **Load average of 8 on 4 cores?** 2× oversubscribed — CPU or I/O saturation.
8. **Find which process uses port 443?** `ss -tlnp | grep :443` or `lsof -i :443`.
9. **Many `CLOSE_WAIT` sockets?** Application isn't closing connections — leak.
10. **`dig` works, app can't resolve?** App uses NSS → check `/etc/hosts`, `nsswitch.conf`, `getent hosts`.
11. **Ping fails but service works?** ICMP blocked; test with `nc -zv`.
12. **Keep a job running after SSH disconnect?** `tmux`/`screen`, `nohup`, or make it a systemd service.
13. **Check cert expiry of a remote host?** `openssl s_client ... | openssl x509 -noout -dates`.

---

## 30. Cheat Sheet

```bash
# SERVICES
systemctl start|stop|restart|reload|status|enable --now|disable|mask|is-active|--failed|cat|edit|daemon-reload
journalctl -u svc -f -n 100 --since "1h ago" -p err -b -1 -k
systemctl list-timers | crontab -e -l | systemd-analyze blame

# PROCESSES
ps aux --sort=-%cpu | ps -ef | pgrep -a | pstree -p | top -o %MEM -H | htop
kill -15/-9/-HUP | pkill -f | killall | timeout
& jobs fg bg nohup disown tmux
nice -n | renice | ionice -c3 | taskset
lsof -p | -i :PORT | +L1 | fuser -vm | strace -fp -c
vmstat 1 | iostat -xz 1 | mpstat -P ALL 1 | pidstat 1 | sar -f | free -h | dmesg -T

# NETWORK
ip -br a | ip r | ip route get IP | ip -s link | ip neigh
ss -tulnp | ss -tan state established | ss -s
ping -c | mtr -rw | traceroute -T | nc -zv host port | echo > /dev/tcp/h/p
dig +short | dig @srv | dig +trace | dig -x | getent hosts | resolvectl
curl -Iv -w timing --resolve -L -k | wget -c
nmcli con mod/up | netplan try/apply | hostnamectl
firewall-cmd --list-all --permanent --add-port | ufw allow | iptables -L -n -v
tcpdump -i any -nn port X -w f.pcap | ethtool -S | iperf3
openssl s_client -connect h:443 -servername h | x509 -noout -dates
sysctl -w | /etc/sysctl.d/ | sysctl --system
```
