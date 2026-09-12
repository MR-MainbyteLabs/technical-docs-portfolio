# SSH Hardening SOP for Linux Lab Environments

**Author:** Michael Rivera | MainbyteLabs
**Version:** 1.0
**Last Updated:** 2024

---

## Overview

This SOP covers SSH hardening for Linux systems in electronics lab and hardware team environments — workstations, headless nodes, Raspberry Pi bench controllers, and any machine you access remotely over SSH.

Default SSH configurations are not production-safe. This procedure closes the common attack vectors: root login, password authentication, weak ciphers, and unnecessary port exposure. Follow it once per machine after initial OS setup, before putting any system on a network with external exposure.

**Time to complete:** 20–30 minutes per system.

---

## Who This Is For

- Electronics lab technicians managing Linux bench nodes or data loggers
- Linux sysadmins standing up new machines in a lab or shop environment
- Anyone running SSH-accessible equipment (Raspberry Pi, BeagleBone, headless workstations) in a small team

This document assumes you already have SSH working. It does not cover initial SSH setup from scratch.

---

## Prerequisites

- Root or sudo access on the target machine
- An existing SSH key pair on your local machine (if you don't have one, see the key generation step below)
- SSH currently working with password authentication (you'll disable this at the end)
- A second terminal session open and connected before making changes — required to avoid locking yourself out

**Critical:** Keep one active SSH session open throughout this entire procedure. Do not close it until you have verified the new configuration works from a second session.

---

## Step-by-Step Guide

### Step 1 — Generate an SSH Key Pair (Skip if You Have One)

On your local machine (not the server):

```bash
ssh-keygen -t ed25519 -C "mainbytelabs-lab-key"
```

- When prompted for a file path, accept the default (`~/.ssh/id_ed25519`) or specify a custom name
- Set a passphrase — do not leave it blank
- This generates two files: `id_ed25519` (private, never share) and `id_ed25519.pub` (public, copied to servers)

---

### Step 2 — Copy Your Public Key to the Target Machine

From your local machine:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub your_user@target_ip
```

Verify it works before moving on:

```bash
ssh -i ~/.ssh/id_ed25519 your_user@target_ip
```

You should log in without being prompted for a password. If it asks for a password, the key copy failed — do not proceed until this works.

---

### Step 3 — Back Up the Current SSH Configuration

On the target machine:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Verify the backup:

```bash
ls -la /etc/ssh/sshd_config*
```

You should see both `sshd_config` and `sshd_config.bak`. If you break something, restore with:

```bash
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
```

---

### Step 4 — Edit the SSH Configuration

Open the config file:

```bash
sudo nano /etc/ssh/sshd_config
```

Apply each of the following changes. Find the existing line and edit it, or add it if it doesn't exist. Remove the `#` comment prefix if present.

#### 4.1 — Change the Default Port

```
Port 2222
```

Use any unused port between 1024–65535. `2222` is common for labs. Note whatever you choose — you will need it for every future connection.

Why: eliminates the majority of automated brute-force attempts that target port 22.

#### 4.2 — Disable Root Login

```
PermitRootLogin no
```

Why: root login over SSH means a successful brute-force attack immediately gives full system access. Never leave this enabled.

#### 4.3 — Disable Password Authentication

```
PasswordAuthentication no
```

**Only set this after Step 2 is confirmed working.** If you set this before your key is in place, you will be locked out.

#### 4.4 — Disable Empty Passwords

```
PermitEmptyPasswords no
```

#### 4.5 — Limit Authentication Attempts

```
MaxAuthTries 3
```

#### 4.6 — Set Login Grace Time

```
LoginGraceTime 30
```

30 seconds to complete authentication before the connection is dropped.

#### 4.7 — Restrict to Specific Users (Optional but Recommended)

```
AllowUsers your_username
```

Replace `your_username` with your actual username. If multiple users need SSH access, space-separate them: `AllowUsers alice bob`.

#### 4.8 — Disable Unused Authentication Methods

```
PubkeyAuthentication yes
KerberosAuthentication no
GSSAPIAuthentication no
UsePAM yes
```

#### 4.9 — Set Secure Ciphers and MACs

Add these lines at the end of the file:

```
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,diffie-hellman-group14-sha256
```

Why: removes legacy ciphers (3DES, arcfour, CBC-mode AES) that have known weaknesses.

#### 4.10 — Disable X11 Forwarding (Unless You Need It)

```
X11Forwarding no
```

If you use GUI applications over SSH, keep this as `yes`.

#### 4.11 — Set Idle Timeout

```
ClientAliveInterval 300
ClientAliveCountMax 2
```

Disconnects idle sessions after 10 minutes (300 seconds × 2 checks). Adjust for your workflow.

---

### Step 5 — Validate the Configuration

Before restarting SSHD, validate the config file for syntax errors:

```bash
sudo sshd -t
```

No output = no errors. If you see errors, review the lines you changed against Step 4.

---

### Step 6 — Restart the SSH Service

```bash
sudo systemctl restart sshd
```

Check that it started cleanly:

```bash
sudo systemctl status sshd
```

Look for `active (running)`. If it shows `failed`, your config has an error — check `sudo journalctl -u sshd` for the specific line.

---

### Step 7 — Test from a Second Terminal (Do Not Close First Session Yet)

Open a new terminal on your local machine. Connect using the new port:

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 your_user@target_ip
```

Verify:
- You connect successfully
- You are NOT prompted for a password
- You land as the correct user (not root)

If the connection works, the hardening is complete. You can now close your original session.

If the connection fails, use your original session to troubleshoot or restore from backup.

---

### Step 8 — Update Firewall Rules

If you are running `ufw`:

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp
sudo ufw status
```

If you are running `firewalld`:

```bash
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload
```

Verify port 22 is no longer listed and port 2222 is open.

---

### Step 9 — Update Your Local SSH Config (Recommended)

On your local machine, edit `~/.ssh/config` (create it if it doesn't exist):

```
Host lab-node-1
    HostName target_ip
    User your_username
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

After this, you connect with:

```bash
ssh lab-node-1
```

Add an entry for every machine you manage. This eliminates typing ports and key paths every time.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Connection refused` after restart | Wrong port in firewall or wrong port in test command | Verify `Port` in sshd_config matches your `ufw allow` rule and your `-p` flag |
| `Permission denied (publickey)` | Key not in `~/.ssh/authorized_keys` on server | Re-run `ssh-copy-id`, verify `~/.ssh/authorized_keys` exists and contains your public key |
| `Permission denied (publickey)` with key present | Wrong file permissions on `~/.ssh` | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| SSHD fails to start | Config syntax error | Run `sudo sshd -t` to identify the broken line, restore backup if needed |
| Locked out completely | Closed original session before verifying | Physical/console access to the machine; restore sshd_config.bak |

---

## Maintenance

- **Rotate keys annually** or any time a team member with key access leaves
- **Review `AllowUsers`** any time team members change
- **Check SSHD logs** monthly: `sudo journalctl -u sshd | grep "Failed\|Invalid" | tail -50`
- **Keep OpenSSH updated**: `sudo apt update && sudo apt upgrade openssh-server` (Debian/Ubuntu)

---

## Related Tools

- [`sftp-ultra`](https://github.com/BleedingCodes/sftp-ultra) — production SFTP engine with SHA-256 verification and SQLite journal; designed for SSH-based file transfer in lab environments
- [`security-scanner`](https://github.com/BleedingCodes/security-scanner) — scans local files for exposed credentials before they leave the machine

---

*Built by MainbyteLabs — technical documentation and Python tooling for electronics labs and hardware teams.*
*https://github.com/MR-MainbyteLabs*
