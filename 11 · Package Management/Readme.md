# 11 · Package Management

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Install, update, pin, roll back, patch and audit software on RHEL-family and Debian-family servers — safely, repeatably, and at fleet scale.

---

## Table of Contents

1. [Package Management Landscape](#1-package-management-landscape)
2. [DNF / YUM (RHEL, CentOS, Rocky, Amazon Linux, Fedora)](#2-dnf--yum)
3. [RPM — Low-Level (RHEL family)](#3-rpm--low-level)
4. [APT (Ubuntu, Debian)](#4-apt-ubuntu-debian)
5. [DPKG — Low-Level (Debian family)](#5-dpkg--low-level)
6. [Side-by-Side Command Map](#6-side-by-side-command-map)
7. [Repositories & GPG Keys](#7-repositories--gpg-keys)
8. [Version Pinning / Locking](#8-version-pinning--locking)
9. [History & Rollback](#9-history--rollback)
10. [Security Patching](#10-security-patching)
11. [Kernel Updates & Reboots](#11-kernel-updates--reboots)
12. [Other Package Tools: snap, flatpak, zypper, apk, pacman](#12-other-package-tools)
13. [Language Package Managers: pip, npm, go, binaries](#13-language-package-managers)
14. [Offline / Air-Gapped & Local Repos](#14-offline--air-gapped--local-repos)
15. [Building From Source](#15-building-from-source)
16. [Troubleshooting](#16-troubleshooting)
17. [Patching at Scale — Best Practices](#17-patching-at-scale--best-practices)
18. [Interview Questions](#18-interview-questions)
19. [Cheat Sheet](#19-cheat-sheet)

---

## 1. Package Management Landscape

| Family | Distros | High-level | Low-level | Format |
|--------|---------|------------|-----------|--------|
| **Red Hat** | RHEL, Rocky, Alma, CentOS Stream, Amazon Linux 2023, Fedora | `dnf` (`yum` on RHEL 7 / AL2) | `rpm` | `.rpm` |
| **Debian** | Ubuntu, Debian | `apt` / `apt-get` | `dpkg` | `.deb` |
| **SUSE** | SLES, openSUSE | `zypper` | `rpm` | `.rpm` |
| **Alpine** | Alpine (containers) | `apk` | — | `.apk` |
| **Arch** | Arch | `pacman` | — | `.pkg.tar.zst` |

> **High-level** tools resolve dependencies and talk to repositories. **Low-level** tools install a single local file with no dependency resolution.

```bash
cat /etc/os-release       # Know which family before running anything
```

---

## 2. DNF / YUM

> On RHEL 8+/AL2023, `yum` is a symlink to `dnf` — same options.

### Install / remove

```bash
sudo dnf install nginx                         # Install
sudo dnf install -y nginx git jq               # Non-interactive (scripts)
sudo dnf install nginx-1.24.0                  # Specific version
sudo dnf install ./app-1.2.rpm                 # Local RPM + resolve deps
sudo dnf reinstall nginx                       # Reinstall (fix corrupted files)
sudo dnf remove nginx                          # Remove
sudo dnf autoremove                            # Remove unused dependencies
sudo dnf downgrade nginx                       # Go to previous version
```

### Update

```bash
sudo dnf check-update                          # List available updates (exit 100 = updates exist)
sudo dnf update                                # Update everything (= upgrade)
sudo dnf update nginx                          # Update one package
sudo dnf update --security                     # Security updates only ← patching
sudo dnf update --exclude=kernel*              # Skip kernel
sudo dnf update --downloadonly --downloaddir=/tmp/pkgs
sudo dnf upgrade-minimal --security            # Minimal version that fixes CVEs
```

### Search / info

```bash
dnf search nginx                               # Search names/summaries
dnf info nginx                                 # Details
dnf list installed                             # All installed
dnf list installed | grep -i java
dnf list --available nginx --showduplicates    # All versions available
dnf provides /usr/bin/netstat                  # Which package gives this file ← very useful
dnf provides '*/semanage'
dnf repoquery -l nginx                         # Files in a package
dnf repoquery --requires nginx                 # Dependencies
dnf repoquery --whatrequires openssl           # Reverse dependencies
dnf deplist nginx
```

### Repos, groups, modules, cache

```bash
dnf repolist                                   # Enabled repos
dnf repolist all                               # All repos
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf config-manager --set-enabled crb
sudo dnf install epel-release                  # Extra Packages (Rocky/Alma)
sudo dnf --enablerepo=epel install htop        # Use repo for one command
sudo dnf --disablerepo="*" --enablerepo=local install app
dnf group list
sudo dnf group install "Development Tools"
dnf module list nodejs
sudo dnf module enable nodejs:20 && sudo dnf install nodejs
sudo dnf clean all                             # Clear cache
sudo dnf makecache                             # Rebuild metadata cache
```

### History

```bash
dnf history                                    # All transactions
dnf history info 42                            # Details of transaction 42
sudo dnf history undo 42                       # Roll back that transaction ← lifesaver
sudo dnf history rollback 40                   # Return to state after transaction 40
```

| Option | What it does |
|--------|--------------|
| `-y` | Assume yes to all prompts |
| `-q` | Quiet, minimal output |
| `--security` | Only security-related updates |
| `--bugfix` | Only bug-fix updates |
| `--exclude=PKG` | Skip packages matching this pattern |
| `--enablerepo=R` | Enable repo for this run |
| `--disablerepo=R` | Disable repo for this run |
| `--showduplicates` | List all versions, not newest |
| `--downloadonly` | Download packages, don't install |
| `--downloaddir=D` | Save downloaded packages here |
| `--nogpgcheck` | Skip signature check ⚠️ avoid |
| `--refresh` | Force metadata refresh before running |
| `--best` | Require newest version or fail |
| `--allowerasing` | Allow removing conflicting packages |
| `--skip-broken` | Skip packages with dependency problems |
| `--assumeno` | Answer no — preview transaction only |

---

## 3. RPM — Low-Level

```bash
rpm -qa                                  # All installed packages
rpm -qa | grep -i openssl
rpm -q nginx                             # Is it installed? Version?
rpm -qi nginx                            # Package info
rpm -ql nginx                            # Files installed by package
rpm -qc nginx                            # Config files only
rpm -qd nginx                            # Doc files
rpm -qf /usr/sbin/nginx                  # Which package owns this file ← debugging
rpm -qR nginx                            # Requirements
rpm -q --changelog openssl | head        # Changelog (CVE fixes)
rpm -qpi app.rpm                         # Info on uninstalled .rpm file
rpm -qpl app.rpm                         # Files in .rpm file
rpm -V nginx                             # Verify files vs package (tamper/drift)
rpm -Va                                  # Verify ALL packages (slow, security audit)
sudo rpm -ivh app.rpm                    # Install (no dep resolution)
sudo rpm -Uvh app.rpm                    # Upgrade or install
sudo rpm -e app                          # Erase
sudo rpm -e --nodeps app                 # ⚠️ Remove ignoring deps
rpm -qa --last | head                    # Recently installed/updated ← what changed?
sudo rpm --import https://repo/key.gpg   # Import GPG key
rpm -K app.rpm                           # Check signature
rpm2cpio app.rpm | cpio -idmv            # Extract without installing
sudo rpm --rebuilddb                     # Fix corrupted RPM DB
```

| Option | What it does |
|--------|--------------|
| `-q` | Query installed package database |
| `-a` | All packages (with -q) |
| `-i` | Info (query) / install (action) |
| `-l` | List files in the package |
| `-c` | List configuration files only |
| `-f FILE` | Which package owns this file |
| `-p FILE` | Query an uninstalled .rpm file |
| `-R` | Show package's requirements/dependencies |
| `-V` | Verify files against package metadata |
| `-U` | Upgrade, or install if missing |
| `-e` | Erase (uninstall) the package |
| `-v` / `-h` | Verbose / show hash progress bar |
| `--last` | Sort by install date, newest first |
| `--nodeps` | Ignore dependencies ⚠️ risky |
| `-K` | Check package signature and digest |

**`rpm -V` output codes:** `S` size · `5` checksum · `T` mtime · `M` mode · `U` user · `G` group · `c` = config file.

---

## 4. APT (Ubuntu, Debian)

```bash
sudo apt update                               # Refresh package index ← ALWAYS FIRST
sudo apt upgrade                              # Upgrade installed (no removals)
sudo apt full-upgrade                         # Upgrade, allow removals (dist-upgrade)
sudo apt install nginx                        # Install
sudo apt install -y nginx=1.24.0-1ubuntu1     # Specific version
sudo apt install ./app_1.2_amd64.deb          # Local .deb + deps
sudo apt install --only-upgrade openssl       # Upgrade only if installed
sudo apt install --no-install-recommends pkg  # Lean installs (Docker images)
sudo apt reinstall nginx
sudo apt remove nginx                         # Remove, keep config
sudo apt purge nginx                          # Remove + config files
sudo apt autoremove --purge                   # Clean unused deps
sudo apt clean                                # Delete all cached .debs
sudo apt autoclean                            # Delete obsolete cached .debs
apt search nginx
apt show nginx                                # Details
apt list --installed
apt list --upgradable                         # Pending updates
apt policy nginx                              # Installed vs candidate version + repo ← useful
apt-cache madison nginx                       # All available versions
apt-cache depends nginx
apt-cache rdepends libssl3                    # Who depends on this
apt-file search /usr/bin/netstat              # Which package provides file (install apt-file)
sudo apt-mark hold nginx                      # Pin current version
sudo apt-mark unhold nginx
apt-mark showhold
sudo apt -f install                           # Fix broken dependencies
```

**Scripts / CI / Dockerfiles** — use `apt-get` (stable output) + non-interactive:
```bash
export DEBIAN_FRONTEND=noninteractive
sudo apt-get update -qq && sudo apt-get install -y -qq --no-install-recommends curl jq
sudo rm -rf /var/lib/apt/lists/*           # Shrink Docker image
```

| Option | What it does |
|--------|--------------|
| `-y` | Assume yes to all prompts |
| `-q` / `-qq` | Quiet / very quiet output |
| `-s` | Simulate, show what would happen |
| `-f` | Fix broken dependencies automatically |
| `--no-install-recommends` | Skip recommended extras, smaller installs |
| `--only-upgrade` | Upgrade only, never fresh install |
| `--reinstall` | Reinstall already installed package |
| `-d` | Download only, don't install |
| `--allow-downgrades` | Permit installing an older version |
| `-o Dpkg::Options::="--force-confold"` | Keep existing config on upgrade |
| `purge` | Remove package and its configs |

---

## 5. DPKG — Low-Level

```bash
dpkg -l                                  # List installed
dpkg -l | grep nginx
dpkg -l nginx                            # Status (ii = installed OK)
dpkg -s nginx                            # Status + details
dpkg -L nginx                            # Files installed by package
dpkg -S /usr/sbin/nginx                  # Which package owns file
dpkg -c app.deb                          # Contents of .deb file
dpkg -I app.deb                          # Info on .deb file
sudo dpkg -i app.deb                     # Install (no dep resolution → then apt -f install)
sudo dpkg -r app                         # Remove
sudo dpkg -P app                         # Purge
sudo dpkg --configure -a                 # Finish interrupted installs ← common fix
dpkg --get-selections | grep hold
sudo dpkg-reconfigure tzdata             # Re-run package config
grep " install " /var/log/dpkg.log | tail   # Install history
```

| Option | What it does |
|--------|--------------|
| `-l` | List packages with status codes |
| `-s` | Show status of a package |
| `-L` | List files owned by package |
| `-S` | Find package owning this file |
| `-i` | Install a local .deb file |
| `-r` / `-P` | Remove / purge including configs |
| `-c` / `-I` | Contents / info of .deb file |
| `--configure -a` | Configure all unpacked, pending packages |

**`dpkg -l` status:** `ii` installed · `rc` removed, config remains · `iU` unpacked, not configured · `iF` failed config.

---

## 6. Side-by-Side Command Map

| Task | RHEL (dnf/rpm) | Ubuntu (apt/dpkg) |
|------|----------------|-------------------|
| Refresh metadata | `dnf makecache` | `apt update` |
| Install | `dnf install pkg` | `apt install pkg` |
| Remove | `dnf remove pkg` | `apt remove` / `purge` |
| Update all | `dnf update` | `apt upgrade` |
| Security only | `dnf update --security` | `unattended-upgrade` / `apt upgrade` from `-security` |
| Search | `dnf search` | `apt search` |
| Info | `dnf info` | `apt show` |
| List installed | `rpm -qa` / `dnf list installed` | `dpkg -l` / `apt list --installed` |
| Files in pkg | `rpm -ql` | `dpkg -L` |
| Owner of file | `rpm -qf` / `dnf provides` | `dpkg -S` / `apt-file search` |
| Available versions | `dnf list --showduplicates` | `apt-cache madison` / `apt policy` |
| Pin version | `dnf versionlock add` | `apt-mark hold` |
| History | `dnf history` | `/var/log/apt/history.log` |
| Rollback | `dnf history undo N` | Install older version manually |
| Clean cache | `dnf clean all` | `apt clean` |
| Repo config | `/etc/yum.repos.d/*.repo` | `/etc/apt/sources.list.d/*` |
| Logs | `/var/log/dnf.log` | `/var/log/apt/`, `/var/log/dpkg.log` |

---

## 7. Repositories & GPG Keys

### RHEL — `/etc/yum.repos.d/myapp.repo`

```ini
[myapp]
name=MyApp Repository
baseurl=https://repo.example.com/rhel/$releasever/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.example.com/RPM-GPG-KEY-myapp
priority=10
```

```bash
sudo dnf config-manager --add-repo URL
sudo dnf config-manager --set-disabled myapp
sudo subscription-manager repos --list-enabled     # RHEL subscriptions
```

### Ubuntu — modern signed-by method

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo add-apt-repository ppa:deadsnakes/ppa       # PPA (Ubuntu only)
```

> `apt-key add` is **deprecated** — use `/etc/apt/keyrings` + `signed-by`.

> Never disable `gpgcheck` in production — it's your protection against tampered packages.

---

## 8. Version Pinning / Locking

Prevent accidental upgrades of critical packages (DB, kernel, container runtime, K8s).

```bash
# RHEL
sudo dnf install python3-dnf-plugin-versionlock
sudo dnf versionlock add kubelet kubeadm kubectl
dnf versionlock list
sudo dnf versionlock delete kubelet
# Or in /etc/dnf/dnf.conf:  exclude=kernel* docker-ce*

# Ubuntu
sudo apt-mark hold kubelet kubeadm kubectl
apt-mark showhold
```

**APT pinning with priorities — `/etc/apt/preferences.d/nginx`**
```
Package: nginx
Pin: version 1.24.*
Pin-Priority: 1001
```

---

## 9. History & Rollback

```bash
# RHEL — true transaction rollback
dnf history list
dnf history info last
sudo dnf history undo last

# Ubuntu — see what changed, then downgrade manually
grep -E "Start-Date|Commandline|Upgrade" /var/log/apt/history.log | tail -20
zcat /var/log/apt/history.log.*.gz | grep -A3 "Start-Date: 2026-09-2"
sudo apt install nginx=1.18.0-6ubuntu14.4 --allow-downgrades

# "What changed on this server recently?" (incident triage)
rpm -qa --last | head -20
ls -lt /var/lib/dpkg/info/*.list | head -20
```

> Best rollback: **VM/EBS snapshot or immutable image** before patching.

---

## 10. Security Patching

### RHEL family

```bash
sudo dnf updateinfo summary              # Count of pending security advisories
sudo dnf updateinfo list --security      # List them
sudo dnf updateinfo list --sec-severity=Critical
sudo dnf updateinfo info RHSA-2026:1234
sudo dnf update --advisory=RHSA-2026:1234   # Apply one advisory
sudo dnf update --cve CVE-2026-12345         # Patch a specific CVE
sudo dnf update --security -y
rpm -q --changelog openssl | grep CVE-2026   # Confirm a CVE is fixed
```

### Ubuntu

```bash
apt list --upgradable 2>/dev/null | grep -i security
sudo unattended-upgrade --dry-run -d
sudo unattended-upgrade                     # Apply security updates
sudo dpkg-reconfigure unattended-upgrades   # Enable automatic
pro status ; pro fix CVE-2026-12345         # Ubuntu Pro
```

### Automatic updates

| Distro | Tool | Config |
|--------|------|--------|
| RHEL | `dnf-automatic` | `/etc/dnf/automatic.conf` → `upgrade_type = security` |
| Ubuntu | `unattended-upgrades` | `/etc/apt/apt.conf.d/50unattended-upgrades` |

```bash
sudo systemctl enable --now dnf-automatic.timer
```

---

## 11. Kernel Updates & Reboots

```bash
uname -r                                    # Running kernel
rpm -q kernel                               # Installed kernels (RHEL)
dpkg -l | grep linux-image                  # Installed kernels (Ubuntu)
sudo grubby --default-kernel                # Default boot kernel (RHEL)
sudo grubby --set-default /boot/vmlinuz-X   # Boot older kernel (rollback)
sudo dnf remove --oldinstallonly --setopt installonly_limit=2 kernel   # Clean old kernels
# Do we need a reboot?
sudo needs-restarting -r                    # RHEL (dnf-utils)  exit 1 = reboot needed
sudo needs-restarting -s                    # Services needing restart
[ -f /var/run/reboot-required ] && cat /var/run/reboot-required      # Ubuntu
sudo needrestart -r l                       # Ubuntu, list services
```

| Option | What it does |
|--------|--------------|
| `needs-restarting -r` | Report if full reboot is required |
| `needs-restarting -s` | List services needing restart |
| `grubby --set-default` | Set kernel used on next boot |
| `installonly_limit=2` | Keep only two kernel versions installed |

> Live patching (no reboot): **kpatch** (RHEL), **Canonical Livepatch** (Ubuntu).

---

## 12. Other Package Tools

```bash
# SUSE
zypper refresh ; zypper install pkg ; zypper update ; zypper se pkg ; zypper patch
# Alpine (containers)
apk update ; apk add --no-cache curl ; apk del curl ; apk info ; apk search
# Arch
pacman -Syu ; pacman -S pkg ; pacman -R pkg ; pacman -Qs pkg
# Snap
snap install kubectl --classic ; snap list ; snap refresh ; snap remove x
# Flatpak (desktops)
flatpak install flathub app ; flatpak list
```

| Option | What it does |
|--------|--------------|
| `apk add --no-cache` | Install without keeping index cache |
| `pacman -Syu` | Sync repos and full upgrade |
| `snap --classic` | Allow full system access confinement |

---

## 13. Language Package Managers

```bash
# Python
python3 -m venv .venv && source .venv/bin/activate      # Always isolate
pip install -r requirements.txt
pip install 'requests==2.32.*'
pip freeze > requirements.txt
pip list --outdated
pipx install ansible-core                               # CLI tools isolated
# Node
npm ci                                                  # Exact lockfile install (CI)
npm install -g pm2
# Go
go install github.com/x/y@latest
# Single binaries (kubectl, terraform, helm)
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -m 0755 kubectl /usr/local/bin/kubectl
```

> Never `sudo pip install` into the system Python — it can break `dnf`/`apt` (which rely on it). Use venv / pipx.

---

## 14. Offline / Air-Gapped & Local Repos

```bash
# RHEL: download with dependencies, then build a repo
sudo dnf download --resolve --destdir=/repo/pkgs nginx
sudo dnf install createrepo_c && createrepo_c /repo/pkgs
sudo dnf reposync --repoid=baseos -p /mirror/        # Mirror a whole repo
# Ubuntu
apt-get download nginx
apt-get install --download-only nginx                # to /var/cache/apt/archives
dpkg-scanpackages . /dev/null | gzip > Packages.gz   # local repo index
```

Enterprise mirrors: **Red Hat Satellite**, **Pulp**, **JFrog Artifactory**, **Sonatype Nexus**, **aptly**.

---

## 15. Building From Source

```bash
sudo dnf groupinstall "Development Tools"     # RHEL
sudo apt install build-essential              # Ubuntu
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
```

> Last resort in production — no updates, no audit trail. Prefer packaging it with `fpm` or `rpmbuild` into your internal repo.

---

## 16. Troubleshooting

| Error | Fix |
|-------|-----|
| `Could not get lock /var/lib/dpkg/lock-frontend` | Another apt running (often unattended-upgrades) → wait; `ps aux \| grep -E 'apt\|dpkg'`. Only remove locks if no process holds them. |
| `dpkg was interrupted` | `sudo dpkg --configure -a` |
| `Unmet dependencies` / broken packages | `sudo apt -f install` |
| `Error: Failed to download metadata for repo` | Network/proxy/DNS, `dnf clean all`, check repo URL/subscription |
| `GPG key retrieval failed` / `NO_PUBKEY` | Import the repo key correctly |
| `Nothing provides X needed by Y` | Enable required repo (EPEL, CRB/PowerTools, AppStream) |
| `Rpmdb checksum is invalid` / DB corruption | `sudo rpm --rebuilddb` |
| `Release file is not valid yet` | Clock wrong → fix NTP (`timedatectl`) |
| `No space left on device` during update | Clean cache (`dnf clean all` / `apt clean`), old kernels, check `/var` & `/boot` |
| Behind corporate proxy | `proxy=http://proxy:8080` in `/etc/dnf/dnf.conf`; `Acquire::http::Proxy "http://proxy:8080";` in `/etc/apt/apt.conf.d/95proxy` |
| `yum` hangs | Stale lock: check `/var/run/yum.pid`, repo timeout, `--disablerepo` slow repo |

---

## 17. Patching at Scale — Best Practices

1. **Snapshot / AMI backup** before patching.
2. **Patch in waves:** dev → staging → canary prod → rest (rolling, keep capacity).
3. **Drain** from load balancer / `kubectl drain` before reboot.
4. **Pin** critical components (kernel, DB, container runtime, kubelet).
5. **Use internal mirrors** so every server gets the exact same versions.
6. **Automate** with Ansible / AWS SSM Patch Manager / Satellite; record results.
7. **Verify** after: service health checks, `needs-restarting`, app smoke tests.
8. **Prefer immutable infra:** bake new images (Packer) and replace, rather than patch in place.

```yaml
# Ansible rolling security patch
- hosts: web
  serial: "25%"
  become: true
  tasks:
    - ansible.builtin.dnf: { name: "*", state: latest, security: true }
    - ansible.builtin.command: needs-restarting -r
      register: reboot_needed
      failed_when: false
      changed_when: reboot_needed.rc == 1
    - ansible.builtin.reboot:
      when: reboot_needed.rc == 1
```

---

## 18. Interview Questions

1. **`rpm` vs `dnf`?** `rpm` installs single files, no deps; `dnf` resolves deps from repos.
2. **Which package provides a file?** `rpm -qf` / `dnf provides` / `dpkg -S`.
3. **`apt remove` vs `apt purge`?** Purge also deletes config files.
4. **How to roll back a bad update on RHEL?** `dnf history undo <id>`.
5. **Prevent a package from upgrading?** `dnf versionlock add` / `apt-mark hold`.
6. **Apply only security patches?** `dnf update --security` / `unattended-upgrade`.
7. **Does the server need a reboot after patching?** `needs-restarting -r` / `/var/run/reboot-required`.
8. **What changed on this box recently?** `rpm -qa --last`, `dnf history`, `/var/log/apt/history.log`.
9. **`apt update` vs `apt upgrade`?** Update refreshes index; upgrade installs newer versions.

---

## 19. Cheat Sheet

```bash
# RHEL
dnf install -y | remove | update --security | check-update | search | info | provides
dnf list --showduplicates | repolist | history | history undo N | versionlock add | clean all
rpm -qa --last | -qi | -ql | -qc | -qf FILE | -V | -ivh | -Uvh | -e | --rebuilddb
# Ubuntu
apt update | upgrade | full-upgrade | install -y --no-install-recommends | remove | purge | autoremove
apt show | policy | list --upgradable | apt-cache madison | apt-mark hold | apt -f install
dpkg -l | -L | -S FILE | -i x.deb | --configure -a
# Reboot?
needs-restarting -r | /var/run/reboot-required
```
