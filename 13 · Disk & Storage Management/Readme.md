# 13 · Disk & Storage Management

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Inspect, partition, format, mount, extend, and troubleshoot storage — from "disk full" alerts to online LVM and cloud volume expansion with zero downtime.

---

## Table of Contents

1. [Storage Stack Overview](#1-storage-stack-overview)
2. [Checking Disk Usage — df, du, ncdu](#2-checking-disk-usage--df-du-ncdu)
3. [Listing Disks & Partitions — lsblk, blkid, fdisk -l](#3-listing-disks--partitions)
4. [Partitioning — fdisk, gdisk, parted](#4-partitioning--fdisk-gdisk-parted)
5. [Filesystems — mkfs & Types](#5-filesystems--mkfs--types)
6. [Mounting — mount, umount, findmnt](#6-mounting--mount-umount-findmnt)
7. [Persistent Mounts — /etc/fstab](#7-persistent-mounts--etcfstab)
8. [LVM — Logical Volume Manager](#8-lvm--logical-volume-manager)
9. [Extending Storage (Cloud & LVM)](#9-extending-storage-cloud--lvm)
10. [Swap](#10-swap)
11. [Filesystem Check & Repair — fsck, xfs_repair](#11-filesystem-check--repair)
12. [Filesystem Tuning & Info — tune2fs, xfs_info, resize](#12-filesystem-tuning--info)
13. [Disk Performance & Health — iostat, iotop, smartctl, fio](#13-disk-performance--health)
14. [RAID — mdadm](#14-raid--mdadm)
15. [Network Storage — NFS, CIFS, iSCSI](#15-network-storage--nfs-cifs-iscsi)
16. [Quotas](#16-quotas)
17. [dd, wipefs & Low-Level Tools](#17-dd-wipefs--low-level-tools)
18. [Encryption — LUKS](#18-encryption--luks)
19. [Containers & Kubernetes Storage](#19-containers--kubernetes-storage)
20. [Incident Playbooks](#20-incident-playbooks)
21. [Interview Questions](#21-interview-questions)
22. [Cheat Sheet](#22-cheat-sheet)

---

## 1. Storage Stack Overview

```
 Application  →  writes /data/file
 Filesystem   →  ext4 / xfs              (mkfs, mount, fstab)
 Logical Vol  →  /dev/vg_data/lv_data    (LVM: pvcreate, vgcreate, lvcreate)
 Partition    →  /dev/nvme1n1p1          (fdisk, gdisk, parted)
 Block Device →  /dev/nvme1n1, /dev/sdb  (physical disk / EBS / Azure disk)
```

| Device naming | Where |
|---------------|-------|
| `/dev/sda`, `/dev/sdb` | SATA/SCSI, VMware, many clouds |
| `/dev/nvme0n1`, `/dev/nvme0n1p1` | NVMe (AWS Nitro, modern servers) |
| `/dev/xvda`, `/dev/xvdf` | Xen (older AWS) |
| `/dev/vda` | KVM/virtio (OpenStack, GCP some) |
| `/dev/mapper/vg-lv` | LVM or encrypted volumes |
| `/dev/md0` | Software RAID |

> Device names can change between boots → always use **UUID** in `/etc/fstab`.

---

## 2. Checking Disk Usage — df, du, ncdu

### df — filesystem-level free space

```bash
df -h                      # Human-readable ← first command on disk alert
df -hT                     # + filesystem type
df -h /var                 # Which FS holds /var and how full
df -i                      # INODE usage ← "disk full" with free space
df -h -x tmpfs -x devtmpfs # Hide virtual filesystems
df -h --output=source,fstype,size,used,avail,pcent,target
```

| Option | What it does |
|--------|--------------|
| `-h` | Human-readable sizes (K, M, G) |
| `-T` | Show filesystem type column |
| `-i` | Show inode usage instead of blocks |
| `-x TYPE` | Exclude this filesystem type |
| `-t TYPE` | Only show this filesystem type |
| `-l` | Only local filesystems, skip network |
| `--output=` | Choose which columns to show |

### du — directory-level usage

```bash
du -sh /var/log                              # Total of one dir
du -sh /var/* 2>/dev/null | sort -hr | head  # Biggest in /var
du -xh / --max-depth=1 2>/dev/null | sort -hr | head -15   # Top-level offenders ← classic
du -ah /var/log | sort -hr | head -20        # Biggest files + dirs
du -sh --exclude='*.gz' /var/log
du --apparent-size -sh file                  # Logical size (sparse files)
```

| Option | What it does |
|--------|--------------|
| `-s` | Summary total only per argument |
| `-h` | Human-readable size output |
| `-a` | Include files, not only directories |
| `-x` | Stay on this filesystem only |
| `-d N` / `--max-depth=N` | Limit recursion to N levels |
| `-c` | Print grand total at end |
| `--exclude=PAT` | Skip files matching pattern |
| `--apparent-size` | Show logical size, not blocks |

### ncdu — interactive (install it on every box)

```bash
ncdu -x /                # Navigate with arrows, 'd' to delete
```

### Find big files

```bash
find / -xdev -type f -size +500M -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr
```

> **`df` and `du` disagree?** → deleted files still open (`lsof +L1`), or files hidden **under a mountpoint**.

---

## 3. Listing Disks & Partitions

```bash
lsblk                         # Tree of disks, partitions, LVM, mountpoints ← start here
lsblk -f                      # + filesystem type, label, UUID
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID,MODEL
lsblk -d                      # Disks only, no partitions
lsblk -p                      # Full device paths
blkid                         # UUIDs and FS types
blkid /dev/nvme1n1p1
sudo fdisk -l                 # Partition tables of all disks
sudo parted -l
cat /proc/partitions
ls -l /dev/disk/by-uuid/ /dev/disk/by-id/
lsscsi                        # SCSI devices
sudo nvme list                # NVMe devices (nvme-cli) — map AWS EBS vol IDs
hwinfo --disk / lshw -class disk
```

| Option | What it does |
|--------|--------------|
| `lsblk -f` | Show filesystem, label, UUID, mount |
| `lsblk -o` | Choose output columns |
| `lsblk -d` | Whole disks only, no partitions |
| `lsblk -p` | Print full /dev device paths |
| `fdisk -l` | List partition tables, don't edit |
| `parted -l` | List partitions on all disks |

**Rescan for a newly attached disk (no reboot)**
```bash
for h in /sys/class/scsi_host/host*/scan; do echo "- - -" | sudo tee $h; done
echo 1 | sudo tee /sys/class/block/sdb/device/rescan      # Detect resized disk
```

---

## 4. Partitioning — fdisk, gdisk, parted

| Table | Max disk | Max partitions | Tool |
|-------|----------|----------------|------|
| **MBR** (msdos) | 2 TB | 4 primary | `fdisk` |
| **GPT** | 9.4 ZB | 128 | `gdisk`, `parted`, modern `fdisk` |

> Use **GPT** for anything new, and always for disks > 2 TB.

### fdisk (interactive)

```bash
sudo fdisk /dev/sdb
#  g  → new GPT table        o → new MBR table
#  n  → new partition        (accept defaults = whole disk)
#  t  → change type (8e / "Linux LVM")
#  p  → print table          d → delete partition
#  w  → write & exit         q → quit without saving
```

| Key | What it does |
|-----|--------------|
| `n` | Create a new partition |
| `d` | Delete an existing partition |
| `p` | Print the current partition table |
| `t` | Change partition type code |
| `g` / `o` | New empty GPT / MBR table |
| `w` | Write changes to disk, exit |
| `q` | Quit without saving any changes |

### parted (scriptable ← automation)

```bash
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s -a optimal /dev/sdb mkpart primary ext4 0% 100%
sudo parted -s /dev/sdb set 1 lvm on
sudo parted /dev/sdb print
sudo parted /dev/sdb resizepart 1 100%
```

| Option | What it does |
|--------|--------------|
| `-s` | Script mode, never prompt |
| `-a optimal` | Align partitions for best performance |
| `mklabel gpt` | Create new GPT partition table |
| `mkpart` | Create partition with start/end |
| `set N lvm on` | Flag partition N as LVM |
| `resizepart N END` | Grow partition N to END |

### Tell the kernel about changes

```bash
sudo partprobe /dev/sdb
sudo partx -u /dev/sdb
```

---

## 5. Filesystems — mkfs & Types

| FS | Shrink? | Grow online? | Best for |
|----|---------|--------------|----------|
| **xfs** | ❌ No | ✅ Yes | RHEL default, large files, DBs, high throughput |
| **ext4** | ✅ Offline | ✅ Yes | Ubuntu default, general purpose |
| **btrfs** | ✅ | ✅ | Snapshots, SUSE default |
| **vfat** | — | — | EFI boot partition |
| **tmpfs** | — | — | RAM-based temp storage |
| **nfs** | — | — | Network shared storage |

```bash
sudo mkfs.xfs /dev/vg_data/lv_data
sudo mkfs.xfs -f -L data /dev/sdb1               # Force + label
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L applogs -m 1 /dev/sdb1         # Label, reserve only 1% for root
sudo mkfs.ext4 -N 2000000 /dev/sdb1              # More inodes (millions of small files)
sudo mkfs -t ext4 /dev/sdb1                      # Generic form
```

| Option | What it does |
|--------|--------------|
| `-L LABEL` | Set filesystem volume label |
| `-f` | xfs: force overwrite existing filesystem |
| `-m N` | ext4: reserved blocks percent for root |
| `-N N` | ext4: set total number of inodes |
| `-i BYTES` | ext4: bytes per inode ratio |
| `-t TYPE` | Filesystem type for generic mkfs |

> ⚠️ `mkfs` **destroys all data** on the target. Triple-check the device with `lsblk -f` first.

---

## 6. Mounting — mount, umount, findmnt

```bash
sudo mkdir -p /data
sudo mount /dev/vg_data/lv_data /data
sudo mount -t xfs /dev/sdb1 /data
sudo mount UUID=1b2c-... /data
sudo mount -o ro /dev/sdb1 /mnt                  # Read-only (forensics/recovery)
sudo mount -o remount,rw /                       # Remount root read-write (rescue)
sudo mount -o remount,noexec /tmp
sudo mount -a                                    # Mount everything in fstab ← test fstab
sudo mount --bind /data/www /var/www             # Bind mount
sudo mount -o loop disk.iso /mnt/iso             # Mount ISO/image
mount | column -t                                # Current mounts
findmnt                                          # Tree view ← best
findmnt /data                                    # Where / what / options
findmnt -T /var/log/app                          # Which mount contains this path
findmnt --verify                                 # Validate /etc/fstab ← before reboot
sudo umount /data
sudo umount -l /data                             # Lazy: detach now, clean when free
sudo umount -f /mnt/nfs                          # Force (stale NFS)
```

| Option | What it does |
|--------|--------------|
| `-t TYPE` | Filesystem type to mount as |
| `-o OPTS` | Comma-separated mount options |
| `-a` | Mount all entries in fstab |
| `--bind` | Mirror a directory at another path |
| `-o loop` | Mount a file as block device |
| `umount -l` | Lazy unmount, detach when not busy |
| `umount -f` | Force unmount, mainly for NFS |
| `findmnt -T` | Find mount containing given path |
| `findmnt --verify` | Check fstab for errors |

**Common mount options**

| Option | What it does |
|--------|--------------|
| `defaults` | rw, suid, dev, exec, auto, async |
| `ro` / `rw` | Read-only / read-write |
| `noexec` | Block running binaries (hardening `/tmp`) |
| `nosuid` | Ignore SUID/SGID bits |
| `nodev` | Ignore device files |
| `noatime` | Skip access-time writes, faster I/O |
| `nofail` | Don't block boot if device missing ← cloud |
| `_netdev` | Wait for network (NFS/iSCSI) |
| `x-systemd.automount` | Mount on first access |
| `discard` | Continuous TRIM for SSDs |

### "target is busy"

```bash
sudo lsof +D /data            # Who has files open
sudo fuser -vm /data          # Processes using mount
cd /                          # Maybe it's YOUR shell!
```

---

## 7. Persistent Mounts — /etc/fstab

```
# <device>                                 <mountpoint> <type> <options>                 <dump> <fsck>
UUID=3f2a1c9e-...                          /data        xfs    defaults,noatime,nofail   0      0
/dev/mapper/vg_app-lv_logs                 /var/log/app ext4   defaults,nofail           0      2
10.0.5.10:/exports/shared                  /mnt/shared  nfs4   defaults,_netdev,nofail   0      0
tmpfs                                      /tmp         tmpfs  defaults,noexec,nosuid,size=2G 0 0
/swapfile                                  none         swap   sw                        0      0
```

| Column | Meaning |
|--------|---------|
| Device | `UUID=` (best), `LABEL=`, or `/dev/mapper/...` |
| Mountpoint | Directory (must exist) |
| Type | xfs, ext4, nfs4, swap, tmpfs |
| Options | Mount options |
| Dump | `0` (legacy backup flag) |
| fsck | `0` skip, `1` root, `2` others (xfs → 0) |

**Safe workflow — never reboot with a broken fstab**
```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%F)
sudo blkid /dev/vg_data/lv_data                   # Get UUID
echo 'UUID=xxxx /data xfs defaults,nofail 0 0' | sudo tee -a /etc/fstab
sudo findmnt --verify                             # Syntax check
sudo mount -a                                     # Must return no errors
sudo systemctl daemon-reload                      # systemd reads fstab into mount units
df -h /data
```

> Broken fstab → boot drops to **emergency mode**. Fix: log in via console, `mount -o remount,rw /`, edit `/etc/fstab`. On AWS: EC2 Serial Console or attach root volume to a rescue instance.

---

## 8. LVM — Logical Volume Manager

```
  Physical Volumes (PV)   /dev/sdb   /dev/sdc
            │                 └─────┬─────┘
  Volume Group (VG)            vg_data  (pool of space)
            │               ┌────────┴────────┐
  Logical Volumes (LV)   lv_app           lv_logs
            │               │                 │
  Filesystem              xfs /app         ext4 /var/log/app
```

**Why LVM:** grow volumes online, span multiple disks, snapshots, move data between disks live.

### Create from scratch

```bash
sudo pvcreate /dev/sdb /dev/sdc                    # 1. Init disks as PVs
sudo vgcreate vg_data /dev/sdb /dev/sdc            # 2. Pool them
sudo lvcreate -n lv_app -L 50G vg_data             # 3. LV of 50 GB
sudo lvcreate -n lv_logs -l 100%FREE vg_data       #    LV using all remaining
sudo mkfs.xfs /dev/vg_data/lv_app                  # 4. Filesystem
sudo mkdir -p /app && sudo mount /dev/vg_data/lv_app /app   # 5. Mount
# 6. Add to /etc/fstab (see §7)
```

### Inspect

```bash
sudo pvs / pvdisplay / pvscan
sudo vgs / vgdisplay / vgscan
sudo lvs / lvdisplay / lvscan
sudo lvs -a -o +devices                     # Which disks each LV lives on
sudo pvs -o +pv_used
```

### Grow

```bash
sudo vgextend vg_data /dev/sdd                     # Add new disk to VG
sudo lvextend -L +20G /dev/vg_data/lv_app          # Grow LV by 20G
sudo lvextend -l +100%FREE /dev/vg_data/lv_app     # Use all free space in VG
sudo lvextend -r -L +20G /dev/vg_data/lv_app       # Grow LV + filesystem in ONE step ← best
# If you didn't use -r, grow FS manually:
sudo xfs_growfs /app                               # xfs (takes MOUNTPOINT)
sudo resize2fs /dev/vg_data/lv_app                 # ext4 (takes DEVICE)
```

### Shrink (ext4 only, offline ⚠️)

```bash
sudo umount /data
sudo e2fsck -f /dev/vg_data/lv_data
sudo lvreduce -r -L 30G /dev/vg_data/lv_data       # -r shrinks FS first
sudo mount /data
```

### Remove / migrate

```bash
sudo pvmove /dev/sdb                               # Move data off a disk (online)
sudo vgreduce vg_data /dev/sdb                     # Remove disk from VG
sudo pvremove /dev/sdb
sudo lvremove /dev/vg_data/lv_old
sudo vgremove vg_old
sudo lvrename vg_data lv_app lv_api
```

### Snapshots (before risky changes / consistent backups)

```bash
sudo lvcreate -s -n lv_app_snap -L 5G /dev/vg_data/lv_app   # Snapshot
sudo mount -o ro,nouuid /dev/vg_data/lv_app_snap /mnt/snap   # nouuid needed for xfs
sudo lvconvert --merge /dev/vg_data/lv_app_snap              # Roll back to snapshot
sudo lvremove /dev/vg_data/lv_app_snap                       # Discard
```

| Command / Option | What it does |
|------------------|--------------|
| `pvcreate` | Initialize disk/partition for LVM use |
| `vgcreate` | Create volume group from PVs |
| `vgextend` | Add physical volume to group |
| `lvcreate -n` | Name of the new logical volume |
| `lvcreate -L 50G` | Exact size for the volume |
| `lvcreate -l 100%FREE` | Use all free extents in VG |
| `lvcreate -s` | Create a snapshot of LV |
| `lvextend -L +20G` | Grow logical volume by 20G |
| `lvextend -r` | Also resize filesystem automatically |
| `lvreduce` | Shrink logical volume ⚠️ data risk |
| `pvmove` | Move extents off disk, online |
| `vgreduce` | Remove PV from volume group |
| `lvconvert --merge` | Revert LV to snapshot state |
| `lvs -o +devices` | Show underlying disks per LV |

---

## 9. Extending Storage (Cloud & LVM)

### Scenario A — Cloud volume enlarged (AWS EBS / Azure / GCP), partition + FS on it

```bash
# 1. Resize the volume in the cloud console / CLI
aws ec2 modify-volume --volume-id vol-0abc --size 200
# 2. Confirm kernel sees new size
lsblk                                     # nvme0n1 now 200G, p1 still 100G
# 3. Grow partition
sudo growpart /dev/nvme0n1 1              # note the SPACE before partition number
# 4. Grow filesystem
sudo xfs_growfs -d /                      # xfs root
sudo resize2fs /dev/nvme0n1p1             # ext4
df -h /
```

### Scenario B — Cloud volume enlarged, LVM on it

```bash
sudo growpart /dev/sdb 1                  # if PV is a partition (skip if whole disk)
sudo pvresize /dev/sdb1                   # PV sees new space
sudo lvextend -r -l +100%FREE /dev/vg_data/lv_app
df -h /app
```

### Scenario C — Add a brand-new disk to an existing LVM

```bash
lsblk                                     # new /dev/sdc
sudo pvcreate /dev/sdc
sudo vgextend vg_data /dev/sdc
sudo lvextend -r -L +100G /dev/vg_data/lv_app
```

| Command | What it does |
|---------|--------------|
| `growpart DISK N` | Grow partition N to fill disk |
| `pvresize` | Update PV size after disk grows |
| `xfs_growfs MOUNT` | Grow xfs to fill its device |
| `resize2fs DEVICE` | Grow or shrink ext2/3/4 filesystem |

> All three scenarios run **online** — no unmount, no downtime (xfs & ext4 grow).

---

## 10. Swap

```bash
swapon --show                     # Active swap
free -h
cat /proc/swaps
# Create a 4G swap file
sudo fallocate -l 4G /swapfile    # (or dd if=/dev/zero of=/swapfile bs=1M count=4096)
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
# Tune
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swap.conf
# Disable
sudo swapoff /swapfile            # (swapoff -a = all; Kubernetes nodes need swap off)
```

| Command / Option | What it does |
|------------------|--------------|
| `swapon --show` | List active swap devices and usage |
| `mkswap` | Format file/partition as swap area |
| `swapon` / `swapoff` | Enable / disable swap space |
| `swapoff -a` | Disable all swap immediately |
| `fallocate -l 4G` | Pre-allocate 4G file instantly |
| `vm.swappiness` | How aggressively kernel uses swap |

---

## 11. Filesystem Check & Repair

> ⚠️ **Never run fsck/repair on a mounted filesystem.** Unmount first (or boot rescue mode for `/`).

```bash
# ext4
sudo umount /data
sudo e2fsck -f /dev/sdb1          # Force full check
sudo e2fsck -fy /dev/sdb1         # Auto-answer yes to fixes
sudo fsck -n /dev/sdb1            # Check only, change nothing
sudo touch /forcefsck             # Check root at next boot (older systems)

# xfs
sudo xfs_repair -n /dev/sdb1      # Dry run
sudo xfs_repair /dev/sdb1
sudo xfs_repair -L /dev/sdb1      # Zero log ⚠️ last resort, may lose recent data
```

| Option | What it does |
|--------|--------------|
| `e2fsck -f` | Force check even if clean |
| `e2fsck -y` | Answer yes to all repairs |
| `fsck -n` | Read-only check, no changes |
| `xfs_repair -n` | No-modify mode, report only |
| `xfs_repair -L` | Clear log, dangerous last resort |

**Filesystem went read-only suddenly?** Kernel remounts `ro` on I/O errors:
```bash
dmesg -T | grep -iE 'error|ext4|xfs|I/O|remount'
```
→ check disk health, repair, then remount.

---

## 12. Filesystem Tuning & Info

```bash
sudo tune2fs -l /dev/sdb1                    # ext4 superblock info
sudo tune2fs -l /dev/sdb1 | grep -iE 'reserved block|inode count|mount count'
sudo tune2fs -m 1 /dev/sdb1                  # Reserved blocks 5% → 1% (frees space instantly!)
sudo tune2fs -L newlabel /dev/sdb1
sudo tune2fs -c 0 -i 0 /dev/sdb1             # Disable periodic fsck
sudo dumpe2fs -h /dev/sdb1
sudo xfs_info /data                          # xfs geometry
sudo xfs_admin -L newlabel /dev/sdb1
sudo e2label /dev/sdb1 applogs
sudo fstrim -av                              # TRIM SSDs (or enable fstrim.timer)
```

| Option | What it does |
|--------|--------------|
| `tune2fs -l` | List ext superblock settings |
| `tune2fs -m N` | Set reserved blocks to N% |
| `tune2fs -L` | Change the filesystem label |
| `tune2fs -c / -i` | Max mount count / interval between checks |
| `fstrim -a` | Trim all mounted filesystems supporting it |
| `fstrim -v` | Report amount of space trimmed |

> ext4 reserves **5%** for root by default — on a 2 TB data disk that's 100 GB. `tune2fs -m 1` on non-root data disks is a quick win during a disk-full emergency.

---

## 13. Disk Performance & Health

```bash
iostat -xz 1                     # Per-device utilization & latency ← key
iotop -oPa                       # Which process is doing I/O
pidstat -d 1                     # Per-process I/O
vmstat 1                         # 'wa' and 'b' columns
sar -d 1 5                       # Disk activity (history with -f)
cat /proc/diskstats
```

**`iostat -xz` columns that matter**

| Column | Meaning | Worry when |
|--------|---------|------------|
| `r/s`, `w/s` | Reads/writes per sec (IOPS) | Near volume IOPS limit |
| `rkB/s`, `wkB/s` | Throughput | Near throughput limit |
| `r_await`, `w_await` | Avg latency ms | SSD > 10ms, HDD > 30ms |
| `aqu-sz` | Queue depth | Consistently > 1–2 |
| `%util` | Busy time | ~100% (less meaningful on NVMe/SSD) |

| Option | What it does |
|--------|--------------|
| `iostat -x` | Extended stats: latency, util, queue |
| `iostat -z` | Hide idle devices with no activity |
| `iotop -o` | Only processes actually doing I/O |
| `iotop -P` | Show processes, not threads |
| `iotop -a` | Accumulated I/O since start |

**Health / benchmark**
```bash
sudo smartctl -H /dev/sda           # Overall health (physical disks)
sudo smartctl -a /dev/sda           # Full SMART attributes
sudo nvme smart-log /dev/nvme0      # NVMe health
sudo fio --name=randread --rw=randread --bs=4k --size=1G --numjobs=4 --iodepth=32 \
         --runtime=60 --time_based --group_reporting --filename=/data/fio.test   # IOPS test
sudo hdparm -tT /dev/sda            # Quick read speed
dd if=/dev/zero of=/data/test bs=1M count=1024 oflag=direct   # Rough write speed
cat /sys/block/nvme0n1/queue/scheduler   # I/O scheduler
```

> AWS gp3/io2 volumes have **IOPS & throughput limits** — high `await` with modest IOPS often = hitting the provisioned limit or EBS burst credits exhausted. Check CloudWatch `VolumeQueueLength`.

---

## 14. RAID — mdadm

| Level | Min disks | Redundancy | Use |
|-------|-----------|------------|-----|
| RAID 0 | 2 | ❌ None | Speed (scratch, ephemeral NVMe striping) |
| RAID 1 | 2 | 1 disk | Mirroring (OS disks) |
| RAID 5 | 3 | 1 disk | Capacity + redundancy |
| RAID 6 | 4 | 2 disks | Large arrays |
| RAID 10 | 4 | 1 per mirror | DBs: speed + redundancy |

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat                          # Status / rebuild progress
sudo mdadm --detail /dev/md0
sudo mdadm --manage /dev/md0 --fail /dev/sdb --remove /dev/sdb
sudo mdadm --manage /dev/md0 --add /dev/sdd
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf   # Persist (/etc/mdadm/mdadm.conf on Ubuntu)
```

| Option | What it does |
|--------|--------------|
| `--create` | Build a new RAID array |
| `--level=N` | RAID level to use |
| `--raid-devices=N` | Number of active member disks |
| `--detail` | Show array status and members |
| `--fail` / `--remove` | Mark disk failed / remove it |
| `--add` | Add replacement or spare disk |
| `--scan` | Detect arrays for config file |

---

## 15. Network Storage — NFS, CIFS, iSCSI

### NFS client

```bash
sudo dnf install nfs-utils   /   sudo apt install nfs-common
showmount -e 10.0.5.10                          # Exports on server
sudo mount -t nfs4 -o rw,hard,timeo=600,retrans=2,_netdev 10.0.5.10:/exports/shared /mnt/shared
nfsstat -m                                      # Mount options in use
# AWS EFS
sudo mount -t efs -o tls fs-0abc:/ /mnt/efs
```

### NFS server

```bash
# /etc/exports
/exports/shared  10.0.0.0/16(rw,sync,no_root_squash)
sudo exportfs -rav                              # Reload exports
```

| Option | What it does |
|--------|--------------|
| `hard` | Retry forever on server failure (safe) |
| `soft` | Give up after retries (data risk) |
| `timeo=N` | Timeout in tenths of a second |
| `_netdev` | Wait for network before mounting |
| `rw,sync` | Export read-write, synchronous writes |
| `root_squash` | Map remote root to nobody (default) |
| `exportfs -rav` | Re-export all, verbose |

**Stale NFS handle / hung `df`:**
```bash
sudo umount -f -l /mnt/shared
df -h -x nfs -x nfs4        # df without hanging on NFS
```

### CIFS/SMB (Windows shares)

```bash
sudo mount -t cifs //fileserver/share /mnt/win -o credentials=/root/.smbcred,uid=app,gid=app,vers=3.0
```

### iSCSI

```bash
sudo iscsiadm -m discovery -t sendtargets -p 10.0.5.20
sudo iscsiadm -m node --login
sudo iscsiadm -m session
```

---

## 16. Quotas

```bash
# fstab option: usrquota,grpquota (ext4)  |  uquota,gquota (xfs)
sudo quotacheck -cug /home && sudo quotaon /home      # ext4
sudo setquota -u manoj 10G 12G 0 0 /home               # soft 10G, hard 12G
sudo repquota -a                                       # Report
quota -u manoj
sudo xfs_quota -x -c 'limit bsoft=10g bhard=12g manoj' /home   # xfs
sudo xfs_quota -x -c 'report -h' /home
```

---

## 17. dd, wipefs & Low-Level Tools

```bash
sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress conv=fsync   # Clone disk to image
sudo dd if=ubuntu.iso of=/dev/sdX bs=4M status=progress oflag=sync        # Write bootable USB
sudo dd if=/dev/zero of=/dev/sdb bs=1M count=10                           # Wipe start of disk
sudo wipefs -a /dev/sdb              # Remove FS/RAID/LVM signatures (reuse a disk)
sudo wipefs /dev/sdb                 # Show signatures only
sudo blockdev --getsize64 /dev/sdb   # Exact size in bytes
sudo sgdisk --zap-all /dev/sdb       # Destroy GPT + MBR tables
```

| Option | What it does |
|--------|--------------|
| `if=` / `of=` | Input file / output file or device |
| `bs=4M` | Block size per read/write |
| `count=N` | Copy only N blocks |
| `status=progress` | Show live transfer progress |
| `conv=fsync` | Flush to disk before finishing |
| `oflag=direct` | Bypass page cache (benchmark) |
| `wipefs -a` | Erase all signatures on device |

> ⚠️ `dd` is nicknamed "disk destroyer" — a swapped `if`/`of` wipes the wrong disk. Verify with `lsblk` every time.

---

## 18. Encryption — LUKS

```bash
sudo cryptsetup luksFormat /dev/sdb1               # Encrypt (destroys data)
sudo cryptsetup open /dev/sdb1 securedata          # Unlock → /dev/mapper/securedata
sudo mkfs.xfs /dev/mapper/securedata
sudo mount /dev/mapper/securedata /secure
sudo cryptsetup close securedata
sudo cryptsetup luksDump /dev/sdb1                 # Header info
sudo cryptsetup luksAddKey /dev/sdb1               # Add another passphrase/key
# /etc/crypttab:  securedata  UUID=xxxx  /root/keyfile  luks
```

> In the cloud, prefer **provider-managed encryption** (EBS/KMS, Azure SSE, GCP CMEK) — transparent, no key handling on the host.

---

## 19. Containers & Kubernetes Storage

```bash
docker system df                         # Images/containers/volumes/cache usage
docker system df -v
docker system prune -af --volumes        # ⚠️ Remove everything unused
docker image prune -a --filter "until=168h"
docker builder prune -af                 # Build cache
du -sh /var/lib/docker/overlay2 /var/lib/containerd 2>/dev/null
find /var/lib/docker/containers -name "*-json.log" -size +500M   # Huge container logs
# Fix at source: /etc/docker/daemon.json → {"log-driver":"json-file","log-opts":{"max-size":"100m","max-file":"3"}}
crictl images ; crictl rmi --prune       # containerd (K8s nodes)
kubectl get pv,pvc -A
kubectl describe pvc data-postgres-0 -n db
kubectl get storageclass
# Kubelet disk pressure → node taint "node.kubernetes.io/disk-pressure", pod evictions
kubectl describe node <node> | grep -A5 Conditions
```

---

## 20. Incident Playbooks

### 🔴 Disk usage alert (e.g., `/` at 95%)

```bash
df -hT                                                  # 1. Which filesystem?
df -i                                                   # 2. Blocks or inodes?
sudo du -xh / --max-depth=1 2>/dev/null | sort -hr | head    # 3. Which dir?
sudo du -xh /var --max-depth=2 2>/dev/null | sort -hr | head
sudo find / -xdev -type f -size +500M -exec ls -lh {} \; 2>/dev/null
sudo lsof +L1                                           # 4. Deleted-but-open files?
# 5. Safe quick wins
sudo journalctl --vacuum-size=500M
sudo dnf clean all || sudo apt clean
sudo find /var/log -name "*.gz" -mtime +7 -delete
: | sudo tee /var/log/app/huge.log                      # Truncate live log (don't rm)
docker system prune -af
# 6. Long-term: logrotate, retention, alerting at 80%, bigger/extra volume
```

### 🔴 `No space left on device` but `df -h` shows free space

```bash
df -i                                    # Inodes 100%!
sudo find / -xdev -type d -size +10M 2>/dev/null     # Dirs with millions of entries
for d in /var/* /tmp; do echo "$(find $d -xdev 2>/dev/null | wc -l) $d"; done | sort -rn | head
# Typical: PHP sessions, mail queue, cache files, K8s/docker leftovers → delete small files
```

### 🔴 Deleted a big file but space not freed

```bash
sudo lsof +L1 | sort -k7 -nr | head      # PID + FD holding deleted file
sudo systemctl restart <service>         # Or truncate via FD:
: | sudo tee /proc/<PID>/fd/<FD>
```

### 🔴 `df` hangs

```bash
mount | grep -E 'nfs|cifs'               # Stale network mount
sudo umount -f -l /mnt/stale
df -h -l                                 # Local filesystems only
```

### 🔴 Server boots into emergency mode

```bash
journalctl -xb | grep -iE 'fstab|mount|fail'
mount -o remount,rw /
vi /etc/fstab                            # Fix UUID / add nofail / comment line
findmnt --verify && mount -a && reboot
```

### 🔴 Filesystem suddenly read-only

```bash
dmesg -T | grep -iE 'I/O error|remount|EXT4-fs error|XFS'
touch /data/testfile                     # Confirm
# Check underlying disk (cloud volume status / smartctl), then unmount + fsck/xfs_repair
```

### 🔴 Disk latency / slow app

```bash
iostat -xz 1                             # High await? queue?
sudo iotop -oPa                          # Which process?
vmstat 1                                 # wa and b columns
# Cloud: check volume IOPS/throughput limits and burst balance
```

---

## 21. Interview Questions

1. **`df` vs `du`?** `df` = filesystem free space; `du` = space used by files/dirs.
2. **Why would `df` show full while `du` shows less?** Deleted-but-open files or data hidden under a mountpoint.
3. **"No space left" with free space?** Inodes exhausted — `df -i`.
4. **Explain PV, VG, LV.** Disks → pooled into a VG → carved into LVs that hold filesystems.
5. **Extend a filesystem online on LVM?** `lvextend -r -L +XG /dev/vg/lv` (or `lvextend` + `xfs_growfs`/`resize2fs`).
6. **After enlarging an EBS volume?** `growpart` → (`pvresize` if LVM) → `xfs_growfs` / `resize2fs`.
7. **Can xfs be shrunk?** No — only grown. ext4 can shrink offline.
8. **Why UUID in fstab?** Device names can change between boots.
9. **Why `nofail` in fstab on cloud servers?** Missing volume won't block boot.
10. **Safely test fstab changes?** `findmnt --verify` and `mount -a` before reboot.
11. **Filesystem turned read-only?** Kernel detected I/O errors — check `dmesg`, disk, run fsck.
12. **`umount: target is busy`?** `lsof +D` / `fuser -vm`, stop processes or `umount -l`.

---

## 22. Cheat Sheet

```bash
# USAGE
df -hT | df -i | du -xh / --max-depth=1 | sort -hr | ncdu -x / | find / -xdev -size +1G | lsof +L1
# DEVICES
lsblk -f | blkid | fdisk -l | parted -l | nvme list | partprobe
# PARTITION + FS
parted -s /dev/sdb mklabel gpt mkpart primary 0% 100% | fdisk (n,t,w)
mkfs.xfs -L | mkfs.ext4 -m 1
# MOUNT
mount -o noatime,nofail | umount -l | findmnt -T | findmnt --verify | mount -a
/etc/fstab: UUID=… /data xfs defaults,nofail 0 0
# LVM
pvcreate | vgcreate | lvcreate -n -L/-l 100%FREE | vgextend | lvextend -r -L +20G
pvs vgs lvs | pvresize | pvmove | lvcreate -s | lvconvert --merge
# GROW (cloud)
growpart /dev/nvme0n1 1 → [pvresize] → xfs_growfs / | resize2fs /dev/…
# SWAP
fallocate -l 4G /swapfile → chmod 600 → mkswap → swapon | vm.swappiness=10
# REPAIR
e2fsck -fy | xfs_repair -n | dmesg -T | tune2fs -l / -m 1
# PERF
iostat -xz 1 | iotop -oPa | smartctl -H | fio
# NET/OTHER
mount -t nfs4 -o hard,_netdev | exportfs -rav | mdadm --detail | cryptsetup open | wipefs -a
docker system df | docker system prune -af
```
