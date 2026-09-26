# 10 · SSH, Key-Based Authentication & Offboarding

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Connect securely to any server, manage keys at scale, harden `sshd`, tunnel through bastions, troubleshoot login failures — and completely remove access when someone leaves.

---

## Table of Contents

1. [How SSH Works](#1-how-ssh-works)
2. [Connecting — `ssh` Command](#2-connecting--ssh-command)
3. [Generating Keys — `ssh-keygen`](#3-generating-keys--ssh-keygen)
4. [Installing Keys on Servers](#4-installing-keys-on-servers)
5. [Required Permissions](#5-required-permissions)
6. [ssh-agent & Agent Forwarding](#6-ssh-agent--agent-forwarding)
7. [Client Config — `~/.ssh/config`](#7-client-config--sshconfig)
8. [Bastion / Jump Hosts](#8-bastion--jump-hosts)
9. [Port Forwarding & Tunnels](#9-port-forwarding--tunnels)
10. [File Transfer: scp, sftp, rsync](#10-file-transfer-scp-sftp-rsync)
11. [Server Hardening — `sshd_config`](#11-server-hardening--sshd_config)
12. [Host Keys & known_hosts](#12-host-keys--known_hosts)
13. [authorized_keys Restrictions](#13-authorized_keys-restrictions)
14. [SSH Certificates (Enterprise Scale)](#14-ssh-certificates-enterprise-scale)
15. [Running Commands on Many Servers](#15-running-commands-on-many-servers)
16. [Troubleshooting SSH](#16-troubleshooting-ssh)
17. [Offboarding — Complete Access Removal](#17-offboarding--complete-access-removal)
18. [Cloud Access Alternatives](#18-cloud-access-alternatives)
19. [Interview Questions](#19-interview-questions)
20. [Cheat Sheet](#20-cheat-sheet)

---

## 1. How SSH Works

```
Client                                         Server (sshd :22)
  │ 1. TCP connect ─────────────────────────────▶ │
  │ 2. ◀── Server host key (verify vs known_hosts)│
  │ 3. Key exchange → encrypted session           │
  │ 4. User auth: public key / password / cert ─▶ │ checks ~/.ssh/authorized_keys
  │ 5. ◀──────────── Shell / command / tunnel     │
```

| Key | Lives where | Shared? |
|-----|-------------|---------|
| **Private key** (`id_ed25519`) | Your laptop only | ❌ Never |
| **Public key** (`id_ed25519.pub`) | Server `~/.ssh/authorized_keys` | ✅ Safe |
| **Host key** (`/etc/ssh/ssh_host_*`) | Server | Identifies the server |

---

## 2. Connecting — `ssh` Command

```bash
ssh manoj@10.0.1.15
ssh -i ~/.ssh/prod.pem ec2-user@ec2-host      # Specific key
ssh -p 2222 manoj@host                        # Custom port
ssh host "uptime; df -h"                      # Run remote command
ssh -t host "sudo systemctl restart nginx"    # Force TTY (sudo prompts)
ssh -J bastion manoj@10.0.2.20                # Via jump host
ssh -v host  /  -vvv host                     # Debug output
ssh -o ConnectTimeout=5 -o BatchMode=yes host # Scripts: fail fast, no prompts
ssh -o StrictHostKeyChecking=accept-new host  # Auto-accept new hosts only
ssh -A bastion                                # Forward agent (careful)
ssh -X host                                   # X11 GUI forwarding
ssh -C host                                   # Compression (slow links)
ssh -N -f -L 5432:db:5432 bastion             # Background tunnel
ssh -q host                                   # Quiet mode
~.                                            # Kill a frozen session (type Enter ~ .)
```

| Option | What it does |
|--------|--------------|
| `-i FILE` | Use this private key file |
| `-p PORT` | Connect to this remote port |
| `-l USER` | Login as this remote user |
| `-v` / `-vvv` | Verbose debug, more v's more detail |
| `-t` | Force pseudo-terminal allocation (sudo/top) |
| `-T` | Disable pseudo-terminal allocation |
| `-J host` | Jump through this bastion host |
| `-A` | Forward your SSH agent to server |
| `-L` | Local port forward to remote |
| `-R` | Remote port forward back to you |
| `-D PORT` | Dynamic SOCKS proxy on port |
| `-N` | No remote command, tunnel only |
| `-f` | Go to background after auth |
| `-C` | Compress all data in session |
| `-X` / `-Y` | Forward X11 (untrusted / trusted) |
| `-q` | Quiet, suppress warnings and banners |
| `-o KEY=VAL` | Set any config option inline |
| `-F FILE` | Use alternate client config file |

**Handy `-o` options**

| Option | What it does |
|--------|--------------|
| `ConnectTimeout=5` | Give up connecting after 5s |
| `BatchMode=yes` | Never prompt; fail instead (scripts) |
| `StrictHostKeyChecking=accept-new` | Accept new keys, reject changed ones |
| `UserKnownHostsFile=/dev/null` | Don't save host keys (ephemeral hosts) |
| `ServerAliveInterval=60` | Keepalive every 60s, avoid drops |
| `IdentitiesOnly=yes` | Only use specified key, not all |
| `PubkeyAuthentication=no` | Force password auth for testing |

---

## 3. Generating Keys — `ssh-keygen`

```bash
ssh-keygen -t ed25519 -C "manoj@bofa-laptop-2026"             # Recommended ← modern
ssh-keygen -t ed25519 -f ~/.ssh/prod_ed25519 -C "prod access"  # Named key
ssh-keygen -t rsa -b 4096 -C "legacy systems"                  # RSA for old systems
ssh-keygen -t ecdsa-sk                                         # Hardware key (YubiKey)
ssh-keygen -p -f ~/.ssh/id_ed25519                             # Change/add passphrase
ssh-keygen -y -f ~/.ssh/id_ed25519 > id_ed25519.pub            # Regenerate public from private
ssh-keygen -y -f mykey.pem                                     # Public key from AWS .pem
ssh-keygen -lf ~/.ssh/id_ed25519.pub                           # Fingerprint (SHA256)
ssh-keygen -lf ~/.ssh/authorized_keys                          # Fingerprints of all authorized keys
ssh-keygen -R 10.0.1.15                                        # Remove host from known_hosts
ssh-keygen -F host                                             # Find host in known_hosts
ssh-keygen -A                                                  # Generate missing host keys (server)
```

| Option | What it does |
|--------|--------------|
| `-t TYPE` | Key type: ed25519, rsa, ecdsa |
| `-b BITS` | Key size in bits (RSA 4096) |
| `-C "text"` | Comment to identify key owner |
| `-f FILE` | Output / input key filename |
| `-N "pass"` | Set passphrase non-interactively |
| `-p` | Change passphrase of existing key |
| `-y` | Print public key from private |
| `-l` | Show fingerprint of key file |
| `-E md5` | Fingerprint hash algorithm to display |
| `-R host` | Remove host from known_hosts |
| `-F host` | Search known_hosts for host |
| `-A` | Create all missing host keys |
| `-s CA` | Sign key with CA (certificates) |

| Type | Recommendation |
|------|----------------|
| **ed25519** | ✅ Default choice: fast, small, secure |
| rsa 4096 | ✅ Compatibility with old systems |
| ecdsa | OK |
| dsa / rsa 1024 | ❌ Deprecated |

> **Always use a passphrase** on personal keys + `ssh-agent`. Comment should identify **person + device** — makes offboarding audits easy.

---

## 4. Installing Keys on Servers

```bash
ssh-copy-id manoj@host                              # Easiest
ssh-copy-id -i ~/.ssh/prod_ed25519.pub -p 2222 manoj@host
# Manual (when ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh manoj@host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

| Option | What it does |
|--------|--------------|
| `-i FILE` | Public key file to install |
| `-p PORT` | SSH port of the server |
| `-f` | Force install, skip existence check |
| `-n` | Dry run, show what's installed |

> At scale, keys are pushed via **Ansible** (`authorized_key` module), **cloud-init**, or central identity (SSSD/LDAP, certificates), not by hand.

---

## 5. Required Permissions

SSH **refuses keys** if permissions are too open (`StrictModes yes`).

| Path | Perms | Owner |
|------|-------|-------|
| `~` (home) | 755 or stricter, **not group/world-writable** | user |
| `~/.ssh/` | **700** | user |
| `~/.ssh/authorized_keys` | **600** | user |
| `~/.ssh/id_ed25519` (private) | **600** | user |
| `~/.ssh/id_ed25519.pub` | 644 | user |
| `~/.ssh/config` | 600 | user |
| `*.pem` | **400** | user |

```bash
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys ~/.ssh/id_* ~/.ssh/config && chmod 644 ~/.ssh/*.pub
chown -R $USER:$USER ~/.ssh
restorecon -Rv ~/.ssh                  # RHEL SELinux context fix
```

---

## 6. ssh-agent & Agent Forwarding

```bash
eval "$(ssh-agent -s)"                  # Start agent
ssh-add ~/.ssh/id_ed25519               # Add key (asks passphrase once)
ssh-add -t 8h ~/.ssh/prod_ed25519       # Auto-expire after 8 hours
ssh-add -l                              # List loaded keys
ssh-add -D                              # Remove all keys
ssh-add -d ~/.ssh/prod_ed25519          # Remove one
ssh-add --apple-use-keychain ~/.ssh/id_ed25519   # macOS keychain
```

| Option | What it does |
|--------|--------------|
| `-l` | List fingerprints of loaded keys |
| `-L` | List full public keys loaded |
| `-D` | Delete all identities from agent |
| `-d` | Delete one identity from agent |
| `-t TIME` | Key lifetime before auto removal |
| `-x` / `-X` | Lock / unlock agent with password |

> ⚠️ **Agent forwarding (`-A`) risk:** root on the remote host can use your agent socket to reach other servers as you. Prefer **`ProxyJump` (`-J`)** — keys never leave your laptop.

---

## 7. Client Config — `~/.ssh/config`

```sshconfig
# Defaults for everything
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes
    IdentitiesOnly yes
    HashKnownHosts yes
    StrictHostKeyChecking accept-new
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 10m

Host bastion
    HostName bastion.prod.example.com
    User manoj
    IdentityFile ~/.ssh/prod_ed25519

Host prod-web-*
    User ec2-user
    IdentityFile ~/.ssh/prod_ed25519
    ProxyJump bastion

Host prod-web-01
    HostName 10.0.2.11

Host db-tunnel
    HostName 10.0.3.50
    ProxyJump bastion
    LocalForward 5432 localhost:5432
```

Now just: `ssh prod-web-01` · `ssh -N db-tunnel`

| Directive | What it does |
|-----------|--------------|
| `Host` | Alias/pattern these settings apply to |
| `HostName` | Real hostname or IP address |
| `User` | Default remote username |
| `Port` | Default remote SSH port |
| `IdentityFile` | Private key to use for host |
| `IdentitiesOnly yes` | Only offer the configured key |
| `ProxyJump` | Connect through this bastion first |
| `LocalForward` | Auto-create local port tunnel |
| `ServerAliveInterval` | Send keepalive every N seconds |
| `ControlMaster auto` | Reuse one connection for many sessions |
| `ControlPersist 10m` | Keep master connection open 10 minutes |
| `StrictHostKeyChecking` | How to handle unknown host keys |
| `ForwardAgent` | Forward agent (keep `no` by default) |

---

## 8. Bastion / Jump Hosts

```bash
ssh -J manoj@bastion manoj@10.0.2.20                # One hop
ssh -J bastion1,bastion2 target                     # Multi-hop
scp -J bastion file.txt 10.0.2.20:/tmp/             # scp through bastion
rsync -e "ssh -J bastion" -avz dir/ 10.0.2.20:/tmp/
# Older OpenSSH (< 7.3)
ssh -o ProxyCommand="ssh -W %h:%p bastion" target
```

---

## 9. Port Forwarding & Tunnels

### Local forward `-L` — reach a private service from your laptop

```bash
ssh -N -L 5432:db.internal:5432 bastion      # localhost:5432 → db.internal:5432
psql -h localhost -p 5432 -U app
ssh -N -L 8080:localhost:80 web01            # Private admin UI on web01
ssh -N -L 9090:prometheus:9090 -L 3000:grafana:3000 bastion   # Multiple
```

### Remote forward `-R` — expose your local port on the remote side

```bash
ssh -N -R 9000:localhost:3000 host           # host:9000 → your laptop:3000
```

### Dynamic `-D` — SOCKS proxy

```bash
ssh -N -D 1080 bastion                        # Browser SOCKS5 → localhost:1080
curl --socks5-hostname localhost:1080 http://internal-app
```

```
 -L  local_port : target_host : target_port     (target resolved from SSH server)
 -R  remote_port : local_host : local_port
```

---

## 10. File Transfer: scp, sftp, rsync

```bash
scp file.txt manoj@host:/tmp/
scp manoj@host:/var/log/app.log .
scp -r ./configs host:/etc/app/
scp -P 2222 -i key.pem file host:~
scp -3 host1:/f host2:/f                  # Copy between two remotes via you
sftp manoj@host                            # Interactive: ls, cd, get, put, mget
sftp -b batch.txt host                     # Scripted sftp
rsync -avzP -e "ssh -p 2222" dir/ host:/opt/dir/     # Preferred for big/repeat copies
```

| Option | What it does |
|--------|--------------|
| `scp -r` | Copy directories recursively |
| `scp -P` | Port (capital P for scp!) |
| `scp -p` | Preserve modification times and modes |
| `scp -C` | Compress during transfer |
| `scp -3` | Route remote-to-remote copy through local |
| `sftp -b` | Run commands from batch file |

---

## 11. Server Hardening — `sshd_config`

File: `/etc/ssh/sshd_config` (+ drop-ins `/etc/ssh/sshd_config.d/*.conf`)

```sshd
Port 22
Protocol 2
PermitRootLogin no                    # No direct root
PasswordAuthentication no             # Keys only
KbdInteractiveAuthentication no       # (ChallengeResponseAuthentication on old versions)
PubkeyAuthentication yes
PermitEmptyPasswords no
AuthenticationMethods publickey       # Or "publickey,keyboard-interactive" for MFA
MaxAuthTries 3
MaxSessions 10
LoginGraceTime 30
ClientAliveInterval 300               # Idle timeout
ClientAliveCountMax 2
AllowGroups ssh-users sre             # Whitelist groups ← strong control
# AllowUsers manoj deploy@10.0.0.*
X11Forwarding no
AllowTcpForwarding no                 # Enable only on bastions if needed
AllowAgentForwarding no
PermitTunnel no
UseDNS no                             # Faster logins
LogLevel VERBOSE                      # Logs key fingerprint per login (audit)
Banner /etc/issue.net
StrictModes yes
UsePAM yes
# Strong crypto (OpenSSH 8+)
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Per-user/group overrides
Match Group sftp-only
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

**Apply safely — never lock yourself out**
```bash
sudo sshd -t                         # Syntax test ← always
sudo sshd -T | grep -i passwordauth  # Show effective config
sudo systemctl reload sshd           # Reload (existing sessions stay)
# Keep your current session OPEN, test login from a SECOND terminal before logging out
```

| Command | What it does |
|---------|--------------|
| `sshd -t` | Test config syntax, report errors |
| `sshd -T` | Print full effective configuration |
| `sshd -T -C user=x,host=y,addr=z` | Effective config for a Match |
| `systemctl reload sshd` | Apply config without dropping sessions |

> Service name: `sshd` on RHEL, `ssh` on Ubuntu. Ubuntu 22.10+ may use socket activation (`ssh.socket`) — port changes go there.

**Brute-force protection:** `fail2ban`, security groups/firewall restricting port 22 to VPN/bastion CIDRs.

```bash
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip 1.2.3.4
```

---

## 12. Host Keys & known_hosts

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
```

**Causes:** server rebuilt, IP reused (cloud/autoscaling), or a man-in-the-middle.

```bash
ssh-keygen -R 10.0.1.15                        # Remove stale entry (after verifying!)
ssh-keyscan -t ed25519 host >> ~/.ssh/known_hosts   # Pre-populate (CI pipelines)
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub    # Server fingerprint (verify out-of-band)
```

| Option | What it does |
|--------|--------------|
| `ssh-keyscan -t` | Fetch only this key type |
| `ssh-keyscan -p` | Scan host on custom port |
| `ssh-keyscan -H` | Hash hostnames in output |

---

## 13. authorized_keys Restrictions

Lock down what a key can do — great for automation keys.

```text
from="10.0.0.0/8",no-agent-forwarding,no-port-forwarding,no-X11-forwarding,no-pty ssh-ed25519 AAAA... backup@jenkins
command="/usr/local/bin/backup.sh",restrict ssh-ed25519 AAAA... backup-only
expiry-time="20261231",restrict ssh-ed25519 AAAA... vendor-temp
```

| Option | What it does |
|--------|--------------|
| `from="CIDR"` | Only allow from these source IPs |
| `command="cmd"` | Force this command, ignore requested one |
| `restrict` | Disable all forwarding, pty, extras |
| `no-pty` | Deny interactive terminal allocation |
| `no-port-forwarding` | Block any tunnel through this key |
| `expiry-time="YYYYMMDD"` | Key stops working after date |

**Central key location (easier audits/offboarding):**
```sshd
AuthorizedKeysFile /etc/ssh/authorized_keys/%u .ssh/authorized_keys
```

---

## 14. SSH Certificates (Enterprise Scale)

Instead of copying public keys to 1000s of servers, a **CA signs short-lived certs**.

```bash
# CA signs a user key, valid 8 hours, principal "manoj"
ssh-keygen -s ca_key -I manoj@corp -n manoj,sre -V +8h ~/.ssh/id_ed25519.pub
# Server trusts the CA:
TrustedUserCAKeys /etc/ssh/user_ca.pub          # in sshd_config
RevokedKeys /etc/ssh/revoked_keys
ssh-keygen -Lf ~/.ssh/id_ed25519-cert.pub       # Inspect certificate
```

| Option | What it does |
|--------|--------------|
| `-s CA` | Sign with this CA private key |
| `-I id` | Certificate identity shown in logs |
| `-n names` | Allowed login principals (usernames) |
| `-V +8h` | Validity window of the certificate |
| `-L` | Print certificate details |

> Tools: **HashiCorp Vault SSH**, **Teleport**, **smallstep**, AWS/GCP OS Login. Offboarding = stop issuing certs; they expire automatically.

---

## 15. Running Commands on Many Servers

```bash
for h in web0{1..5}; do ssh -o ConnectTimeout=5 "$h" 'hostname; uptime'; done
parallel-ssh -h hosts.txt -i "df -h /"                # pssh
pdsh -w web[01-10] "systemctl is-active nginx"
ansible webservers -m shell -a "uptime"               # Ansible ad-hoc ← standard
ansible all -m ping
```

---

## 16. Troubleshooting SSH

**Step 1 — client side verbose**
```bash
ssh -vvv manoj@host 2>&1 | grep -Ei "offering|authentications|denied|refused|debug1: Server"
```

**Step 2 — server side logs**
```bash
sudo tail -f /var/log/secure        # RHEL
sudo tail -f /var/log/auth.log      # Ubuntu
sudo journalctl -u sshd -f
```

**Step 3 — network & service**
```bash
nc -zv host 22                      # Port reachable?
sudo ss -tlnp | grep ssh            # sshd listening?
sudo systemctl status sshd
```

| Symptom | Likely cause → Fix |
|---------|--------------------|
| `Connection timed out` | Firewall / Security Group / NACL / wrong IP or route |
| `Connection refused` | sshd not running or wrong port |
| `Permission denied (publickey)` | Wrong key/user, key not in `authorized_keys`, bad perms, `AllowGroups` |
| `Authentication refused: bad ownership or modes` (server log) | Fix `~`, `~/.ssh`, `authorized_keys` perms |
| `UNPROTECTED PRIVATE KEY FILE` | `chmod 600` (or 400) the key |
| `Too many authentication failures` | Agent offering many keys → `-o IdentitiesOnly=yes -i key` |
| `Host key verification failed` | Changed host key → verify, then `ssh-keygen -R` |
| Login slow (10–30s) | `UseDNS yes` / GSSAPI → `UseDNS no`, `GSSAPIAuthentication no` |
| Session drops when idle | `ServerAliveInterval 60` client-side |
| Account works for others, not you | Account locked/expired → `chage -l`, `faillock` |
| AWS EC2 wrong user | Amazon Linux `ec2-user`, Ubuntu `ubuntu`, RHEL `ec2-user`, Debian `admin` |
| SELinux blocks key (RHEL) | `restorecon -Rv ~/.ssh` |
| `no matching host key type found` | Old server: `-o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa` |

---

## 17. Offboarding — Complete Access Removal

When an engineer leaves or a contractor ends, **remove every path in**, keep evidence, don't break services.

### Checklist

```bash
USER=olduser

# 1. Identify
id $USER; getent passwd $USER
last $USER | head                              # Last logins
who | grep $USER                                # Currently logged in?

# 2. Lock the account (all auth methods, incl. SSH keys)
sudo usermod -L $USER                           # Lock password
sudo usermod -e 1 $USER                         # Expire account → blocks key auth too
sudo usermod -s /sbin/nologin $USER             # No shell

# 3. Kill active sessions & processes
sudo pkill -KILL -u $USER
sudo loginctl terminate-user $USER

# 4. Remove SSH keys
sudo mv /home/$USER/.ssh/authorized_keys /root/offboard_${USER}_authorized_keys.bak
# Also search for their key on OTHER accounts (shared/service accounts!)
FP=$(ssh-keygen -lf /root/offboard_${USER}_authorized_keys.bak | awk '{print $2}')
sudo grep -rl "olduser@" /home/*/.ssh/authorized_keys /root/.ssh/authorized_keys /etc/ssh/authorized_keys 2>/dev/null

# 5. Remove sudo rights
sudo gpasswd -d $USER wheel 2>/dev/null; sudo gpasswd -d $USER sudo 2>/dev/null
sudo grep -rn "$USER" /etc/sudoers /etc/sudoers.d/
sudo rm -f /etc/sudoers.d/$USER && sudo visudo -c

# 6. Remove from all groups
for g in $(id -nG $USER); do sudo gpasswd -d $USER $g; done

# 7. Cron jobs, at jobs, and running services
sudo crontab -l -u $USER && sudo crontab -r -u $USER
sudo atq | grep $USER
systemctl list-units --all | grep -i $USER
sudo grep -rl "User=$USER" /etc/systemd/system/

# 8. Archive home, then (after retention period) delete
sudo tar -czpf /archive/${USER}_home_$(date +%F).tgz /home/$USER
sudo userdel -r $USER                            # Only when approved

# 9. Orphaned files
sudo find / -xdev \( -user $USER -o -nouser \) 2>/dev/null

# 10. Verify
sudo -l -U $USER; sudo chage -l $USER; grep $USER /etc/passwd
```

### Beyond the Linux box

| Area | Action |
|------|--------|
| **Shared secrets** | Rotate passwords/keys the user knew (service accounts, DB, root/break-glass) |
| **Shared SSH keys / .pem** | Replace keys they had copies of (e.g., AWS key pairs) |
| **Bastion / VPN** | Disable VPN, bastion and MFA tokens |
| **Identity provider** | Disable in AD/LDAP/Okta/SSO → cascades via SSSD |
| **Cloud IAM** | Remove IAM users, access keys, roles, OS Login bindings |
| **SSH CA** | Stop issuing certs; add key to `RevokedKeys` |
| **Git / CI/CD** | Remove from GitHub/GitLab, Jenkins, deploy keys, personal tokens |
| **Kubernetes** | Remove RoleBindings, kubeconfig certs/tokens |
| **Config mgmt** | Remove user from Ansible/Puppet/Chef user lists so they aren't re-added |
| **Evidence** | Record ticket #, timestamp, actions for audit (SOX / PCI) |

### At scale — Ansible example

```yaml
- hosts: all
  become: true
  tasks:
    - name: Lock and expire user
      ansible.builtin.user:
        name: olduser
        password_lock: true
        expires: 1
        shell: /sbin/nologin
    - name: Remove authorized key
      ansible.posix.authorized_key:
        user: olduser
        key: "{{ lookup('file', 'keys/olduser.pub') }}"
        state: absent
    - name: Remove sudoers drop-in
      ansible.builtin.file:
        path: /etc/sudoers.d/olduser
        state: absent
```

---

## 18. Cloud Access Alternatives

| Tool | Benefit |
|------|---------|
| **AWS SSM Session Manager** | No port 22, no keys, IAM-based, logged to CloudTrail/S3 |
| **EC2 Instance Connect** | Push a one-time key valid 60s |
| **GCP OS Login / IAP** | IAM-controlled SSH, no key sprawl |
| **Azure Bastion** | Browser-based SSH, no public IP |
| **Teleport / Boundary** | Certificate-based, session recording, just-in-time access |

```bash
aws ssm start-session --target i-0abc123
aws ec2-instance-connect send-ssh-public-key --instance-id i-0abc --instance-os-user ec2-user --ssh-public-key file://id_ed25519.pub
gcloud compute ssh vm-1 --tunnel-through-iap
```

---

## 19. Interview Questions

1. **How does key-based auth work?** Server checks your signature against the public key in `authorized_keys`; private key never leaves client.
2. **`Permission denied (publickey)` — how to debug?** `ssh -vvv`, server auth log, check key/user, perms (700/600), `AllowGroups`, SELinux.
3. **Why ed25519 over RSA?** Smaller, faster, strong security by default.
4. **ProxyJump vs agent forwarding?** ProxyJump keeps keys local; agent forwarding exposes agent to remote root.
5. **Local vs remote port forwarding?** `-L` brings remote service to you; `-R` exposes your service to remote.
6. **How to safely change `sshd_config`?** `sshd -t`, reload, test from a second session before closing first.
7. **User left — how do you revoke access?** Lock + expire, remove keys everywhere, sudo, groups, cron; rotate shared secrets; disable IdP/VPN/IAM; archive; audit.
8. **Why is `passwd -l` not enough?** SSH keys still work — expire the account and remove keys.
9. **`REMOTE HOST IDENTIFICATION HAS CHANGED` — what do you do?** Verify fingerprint out-of-band, then `ssh-keygen -R host`.

---

## 20. Cheat Sheet

```bash
ssh -i key -p port -J bastion -t -v -o ConnectTimeout=5 -o BatchMode=yes user@host
ssh-keygen -t ed25519 -C | -p | -y -f | -lf | -R host
ssh-copy-id -i key.pub user@host
chmod 700 ~/.ssh ; chmod 600 authorized_keys id_* ; chmod 400 *.pem
eval $(ssh-agent) ; ssh-add -t 8h ; ssh-add -l
~/.ssh/config : Host HostName User IdentityFile ProxyJump LocalForward ControlMaster
ssh -N -L 5432:db:5432 bastion | -R | -D 1080
scp -r -P | sftp | rsync -avzP -e ssh
sshd_config : PermitRootLogin no, PasswordAuthentication no, AllowGroups, MaxAuthTries 3
sshd -t ; sshd -T ; systemctl reload sshd ; tail -f /var/log/secure|auth.log
OFFBOARD: usermod -L -e 1 -s /sbin/nologin ; pkill -u ; rm keys ; sudoers ; crontab -r ; archive ; rotate secrets
```
