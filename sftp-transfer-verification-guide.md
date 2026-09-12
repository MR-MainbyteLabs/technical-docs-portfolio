# SFTP Transfer Verification Guide for Linux Lab Environments

**Author:** Michael Rivera | MainbyteLabs
**Version:** 1.0
**Last Updated:** 2024

---

## Overview

This guide covers reliable, verifiable SFTP file transfer in Linux lab environments — including manual verification methods, common failure modes, and automated transfer using [`sftp-ultra`](https://github.com/BleedingCodes/sftp-ultra).

SFTP transfers can silently fail. A file that appears complete may be truncated, corrupted in transit, or partially written due to a dropped connection. In electronics labs, test data, firmware images, and measurement logs require confirmed integrity — not just a successful-looking copy.

This document covers three levels of rigor: basic transfer with manual check, checksum-verified transfer, and automated production transfer with journaling.

---

## Who This Is For

- Electronics lab technicians transferring test data, firmware, or measurement logs between bench nodes and storage
- Linux sysadmins managing file transfer between lab workstations, NAS devices, and remote servers
- Any team that has been burned by corrupted or incomplete file transfers and needs a repeatable verification process

---

## Prerequisites

- SSH access to both source and destination systems (see: SSH Hardening SOP)
- `openssh-client` installed locally: `ssh -V` should return a version string
- `sha256sum` available on both systems: standard on all major Linux distributions
- For automated transfer: Python 3.11+, `sftp-ultra` installed (setup instructions below)

---

## Core Concepts

### Why Transfers Fail Silently

SFTP does not guarantee end-to-end integrity by default. Common failure modes:

- **Truncated files**: connection dropped mid-transfer; destination file exists but is incomplete
- **Partial writes**: disk full on destination; file written up to the point of failure with no error raised to the user
- **Silent corruption**: rare, but possible on unreliable network paths or with faulty storage
- **Interrupted `.part` files**: if your transfer tool uses a staging filename, an interrupted transfer can leave a `.part` file that is mistaken for a complete file

### SHA-256 as the Verification Standard

SHA-256 produces a 64-character hex digest of file content. Two files with identical SHA-256 hashes are, for all practical purposes, identical. Generate on source before transfer, generate on destination after — if they match, the file is intact.

This is not optional for test data or firmware. Make it part of every transfer procedure.

---

## Method 1 — Basic Transfer with Manual SHA-256 Verification

Use this when you are doing a one-off transfer and do not have automated tooling in place.

### Step 1 — Generate Checksum on Source

On the source machine:

```bash
sha256sum /path/to/file.ext
```

Example output:

```
a3f1c2d9e4b7083a1d9e2f4c6b8a0c3e7d5f9b2a4c6e8d1f3b5a7c9e0d2f4b6  file.ext
```

Save this hash. Write it down, copy it to a file, or include it in your transfer ticket. You will compare against it after transfer.

To save it automatically:

```bash
sha256sum /path/to/file.ext > /path/to/file.ext.sha256
```

This creates a sidecar `.sha256` file with the hash and filename. Transfer this file alongside your payload.

---

### Step 2 — Transfer the File

Using the standard `sftp` command:

```bash
sftp -P 2222 user@destination_ip
```

Inside the SFTP session:

```
put /path/to/file.ext /destination/path/file.ext
exit
```

Or in a single non-interactive command:

```bash
sftp -P 2222 user@destination_ip:/destination/path/ <<< "put /path/to/file.ext"
```

If transferring a `.sha256` sidecar file:

```bash
sftp -P 2222 user@destination_ip:/destination/path/ <<< $'put /path/to/file.ext\nput /path/to/file.ext.sha256'
```

---

### Step 3 — Verify on Destination

On the destination machine, generate the hash of the received file:

```bash
sha256sum /destination/path/file.ext
```

Compare to the hash from Step 1. They must match character for character.

If you transferred a `.sha256` sidecar, you can verify automatically:

```bash
cd /destination/path/
sha256sum -c file.ext.sha256
```

Expected output:

```
file.ext: OK
```

Any output other than `OK` means the transfer failed integrity check. Delete the destination file and retransfer.

---

### Step 4 — Verify File Size as a Secondary Check

File size is not a substitute for SHA-256 verification, but a size mismatch is an immediate red flag that catches truncation without running a full hash.

On source:

```bash
stat -c "%s %n" /path/to/file.ext
```

On destination:

```bash
stat -c "%s %n" /destination/path/file.ext
```

Byte counts must be identical. If they differ, the transfer is incomplete regardless of what the transfer client reported.

---

## Method 2 — Resumable Transfer with Safe Staging Pattern

Use this for large files (video, firmware images, large test datasets) on unreliable connections.

### The `.part` File Pattern

Never write directly to the final filename during transfer. Write to a staging file with a `.part` suffix, then rename atomically after transfer and verification complete.

Manual implementation:

```bash
# On destination machine — receive to staging file
sftp user@source_ip:/path/to/largefile.bin /destination/largefile.bin.part

# After transfer completes — verify before renaming
sha256sum /destination/largefile.bin.part
# Compare to source hash manually

# Only rename if hash matches
mv /destination/largefile.bin.part /destination/largefile.bin
```

This pattern guarantees that the final filename only ever contains a verified complete file. Any interrupted transfer leaves a `.part` file that is immediately identifiable as incomplete.

### Resuming an Interrupted Transfer

The standard `sftp` command does not natively support resume. Use `rsync` over SSH for large file resume:

```bash
rsync -avP --partial -e "ssh -p 2222" user@source_ip:/path/to/largefile.bin /destination/largefile.bin.part
```

Flags:
- `-a` — archive mode (preserves permissions, timestamps)
- `-v` — verbose
- `-P` — show progress + keep partial files on interrupt
- `--partial` — keep the partial file on destination for resume

Re-run the same command on interrupt — rsync will resume from where it stopped.

After rsync completes, verify with SHA-256 before renaming from `.part` to final.

---

## Method 3 — Automated Production Transfer with sftp-ultra

[`sftp-ultra`](https://github.com/BleedingCodes/sftp-ultra) automates everything in Methods 1 and 2: concurrent workers, resumable `.part` downloads, SHA-256 verification, SQLite transfer journal, and exponential backoff on connection failure.

Use this when:
- You transfer files regularly between fixed endpoints
- You need a log of every transfer with verification status
- You cannot afford to manually verify every transfer

### Installation

```bash
git clone https://github.com/BleedingCodes/sftp-ultra.git
cd sftp-ultra
pip install -r requirements.txt
```

### Basic Usage

Transfer a single file with verification:

```bash
python sftp_ultra.py \
  --host destination_ip \
  --port 2222 \
  --user your_username \
  --key ~/.ssh/id_ed25519 \
  --remote /source/path/file.bin \
  --local /destination/path/
```

sftp-ultra will:
1. Download to `/destination/path/file.bin.part`
2. Verify SHA-256 against the remote file hash
3. Rename to `/destination/path/file.bin` only if verification passes
4. Log the transfer result to its SQLite journal

### Dry Run

Test what would transfer without moving any files:

```bash
python sftp_ultra.py \
  --host destination_ip \
  --port 2222 \
  --user your_username \
  --key ~/.ssh/id_ed25519 \
  --remote /source/path/ \
  --local /destination/path/ \
  --dry-run
```

### Querying the Transfer Journal

sftp-ultra logs every transfer to a local SQLite database. Query it to audit transfer history:

```bash
sqlite3 sftp_journal.db "SELECT filename, sha256, status, timestamp FROM transfers ORDER BY timestamp DESC LIMIT 20;"
```

Columns:
- `filename` — transferred file
- `sha256` — hash verified at transfer time
- `status` — `VERIFIED`, `FAILED`, or `PARTIAL`
- `timestamp` — UTC timestamp of transfer completion

---

## Transfer Checklist

Use this as a pre/post transfer checklist for critical files (test data, firmware, signed reports):

**Before transfer:**
- [ ] SHA-256 hash generated on source
- [ ] Hash saved to sidecar `.sha256` file or recorded in transfer log
- [ ] Destination path confirmed to exist and have sufficient space

**During transfer:**
- [ ] Transfer writing to `.part` staging file (not final filename)
- [ ] Transfer tool confirmed to be running (not silently failed)

**After transfer:**
- [ ] SHA-256 verified on destination — matches source hash exactly
- [ ] File size matches source (secondary check)
- [ ] `.part` file renamed to final filename only after verification
- [ ] Transfer logged (journal entry or ticket update)

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `sha256sum -c` returns `FAILED` | Transfer corruption or truncation | Delete destination file, retransfer from source |
| Destination file size smaller than source | Interrupted transfer | Check for `.part` file; resume with rsync or retransfer |
| `Connection reset by peer` mid-transfer | Network instability or server timeout | Use rsync `--partial` for resume; check `ClientAliveInterval` on SSH server |
| `sftp: Couldn't canonicalise: No such file or directory` | Destination path does not exist | Create destination directory before transfer: `ssh user@dest "mkdir -p /path"` |
| `.part` file left on destination | Interrupted sftp-ultra transfer | Re-run sftp-ultra — it will resume from the `.part` file |
| Permission denied on destination write | Wrong directory permissions | `chmod 755 /destination/path` or check ownership |

---

## Related Documentation

- [SSH Hardening SOP for Linux Lab Environments](./ssh-hardening-sop.md) — required reading before setting up SFTP between lab nodes
- [`sftp-ultra`](https://github.com/BleedingCodes/sftp-ultra) — automated SFTP with built-in verification and journaling
- [`security-scanner`](https://github.com/BleedingCodes/security-scanner) — scan files for exposed credentials before transferring off-machine

---

*Built by MainbyteLabs — technical documentation and Python tooling for electronics labs and hardware teams.*
*https://github.com/MR-MainbyteLabs*
