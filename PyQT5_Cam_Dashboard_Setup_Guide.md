# PyQt Camera Dashboard — Setup & User Guide

**Document Type:** Technical Setup & User Guide
**Platform:** Linux (Ubuntu/Debian) | Windows 10/11
**Skill Level:** Beginner to Intermediate
**Version:** 1.0
**Last Updated:** 2026

---

## What This Guide Covers

This guide walks through the full setup and daily use of the PyQt Camera Dashboard — a
local multi-camera monitoring and recording application built with Python, OpenCV, and PyQt5.

> **Source code:** [github.com/BleedingCodes/pyqt-camera-dashboard](https://github.com/BleedingCodes/pyqt-camera-dashboard)

By the end of this guide you will have:

- The application installed and running
- Your cameras configured and displaying live feeds
- Recordings saving automatically to an organized folder structure
- An understanding of how the system works under the hood

---

## What the Application Does

The PyQt Camera Dashboard connects to IP cameras over RTSP and displays their live feeds
in a single dark-theme GUI window. It records, reconnects automatically on stream failure,
and manages disk space without manual intervention.

Camera credentials are never stored in plaintext. The app generates an encryption key on
first run and uses it to encrypt the config file after every save.

---

## System Requirements

| Item | Requirement |
|---|---|
| Python | 3.10 or higher |
| Operating System | Linux (Ubuntu/Debian) or Windows 10/11 |
| Camera Protocol | RTSP |
| Network | Camera and host machine on the same local network |
| Disk Space | Depends on camera count and recording duration |

---

## Dependencies

| Package | Purpose |
|---|---|
| `PyQt5` | GUI framework — windows, buttons, dialogs |
| `opencv-python` | RTSP stream capture and video recording |
| `cryptography` | Fernet encryption for credential storage |

---

## Phase 1 — Installation

### Linux / Ubuntu

**Install system packages:**

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv git
```

**Clone the repository:**

```bash
git clone https://github.com/BleedingCodes/pyqt-camera-dashboard.git
cd pyqt-camera-dashboard
```

**Create and activate a virtual environment:**

```bash
python3 -m venv venv
source venv/bin/activate
```

**Install Python dependencies:**

```bash
pip install -r requirements.txt
```

**Run the app:**

```bash
python camera_dashboard.py
```

---

### Windows

**Clone the repository:**

```powershell
git clone https://github.com/BleedingCodes/pyqt-camera-dashboard.git
cd pyqt-camera-dashboard
```

**Create and activate a virtual environment:**

```powershell
py -m venv venv
venv\Scripts\activate
```

**Install dependencies:**

```powershell
pip install -r requirements.txt
```

**Run the app:**

```powershell
python camera_dashboard.py
```

---

## Phase 2 — First-Time Setup

On first launch the app runs a one-time setup wizard before opening the dashboard.

**Step 1 — Key generation**

The app checks for `secret.key` in the script folder. If it does not exist, a new
Fernet encryption key is generated and saved there automatically. This happens silently
with no action required.

**Step 2 — Camera setup wizard**

A dialog appears asking how many cameras you want to monitor. For each camera you will
be prompted to enter:

| Field | Description |
|---|---|
| Camera IP address | The local network IP of the camera |
| Display name | A friendly label shown on the camera tile |
| Username | Camera login username |
| Password | Camera login password (masked during entry) |

After all cameras are entered, the app saves the details to `camera_config.json` and
immediately encrypts it. The dashboard then opens with all configured cameras connecting.

**On every launch after the first**, the app decrypts and loads `camera_config.json`
automatically. The setup wizard does not appear again unless the config file is deleted.

---

## Phase 3 — Using the Dashboard

### Camera Tiles

Each camera is displayed as a tile in a two-column grid. Every tile shows:

- Camera name
- Live video feed
- Current status (Connecting / Live / Recording / Offline)
- Individual control buttons

### Tile Buttons

| Button | What it does |
|---|---|
| Start Recording | Begins recording this camera to disk |
| Stop Recording | Stops recording this camera |
| Reconnect | Manually reconnects this camera without restarting the app |
| Remove | Removes this tile from the current session only — does not delete from config |

### Dashboard Buttons

| Button | What it does |
|---|---|
| + | Opens the add-camera dialog to add a new camera while the app is running |
| Start All | Starts recording on every active camera simultaneously |
| Stop All | Stops recording on every active camera simultaneously |

---

## Phase 4 — Adding a Camera After Setup

To add a camera while the app is already running:

1. Click the **+** button in the bottom control bar
2. Enter the camera IP, display name, username, and password
3. Click OK

The new camera tile appears immediately and begins connecting. The config file is updated
and re-encrypted automatically.

---

## Phase 5 — Recording

### How Recording Works

Each camera records independently. Recordings are split into 15-minute segments
automatically. When one segment reaches the time limit, the current file is closed and
a new one opens without interrupting the stream.

### File Naming

Each recording file is named using the camera's display name and the timestamp when
recording started:

```text
cameraname_YYYY-MM-DD_HH-MM-SS.mp4
```

Example:

```text
front_door_2026-09-05_14-32-10.mp4
```

If two cameras share the same display name, the app automatically appends a number to
keep filenames unique and prevent one recording from overwriting another.

### Folder Structure

Recordings are organized automatically by date and hour:

```text
recordings/
└── 2026-09-05/
    └── 14/
        ├── front_door_2026-09-05_14-32-10.mp4
        └── back_yard_2026-09-05_14-32-10.mp4
```

---

## Phase 6 — Automatic Disk Cleanup

The app monitors disk usage once per hour. When usage on the recordings drive exceeds
**75%**, it deletes the oldest recording files one by one until usage drops back to a
safe level.

No manual cleanup is needed. The process runs silently in the background.

---

## Phase 7 — Camera Reconnection

Each camera runs in its own background thread. If the stream drops for any reason:

1. The worker retries the connection automatically up to **3 times**
2. Between each attempt it waits 3 seconds
3. If all 3 attempts fail, the tile shows **Camera Offline** and enables the Reconnect button
4. Clicking **Reconnect** resets the failure counter and tries again immediately

Reconnecting one camera does not affect any other camera or interrupt recordings in progress.

---

## Security

| File | Purpose | Safe to share? |
|---|---|---|
| `secret.key` | Fernet encryption key | Never — keep private |
| `camera_config.json` | Encrypted camera credentials | Never — keep private |
| `recordings/` | Video footage | Your discretion |

Both `secret.key` and `camera_config.json` are listed in `.gitignore` and will not be
pushed to GitHub accidentally.

> If `secret.key` is lost or deleted, the encrypted `camera_config.json` cannot be
> decrypted. Delete both files and run the setup wizard again to reconfigure.

---

## File Reference

| File | Description |
|---|---|
| `camera_dashboard.py` | Main application — run this to start |
| `camera_config.json` | Encrypted camera credentials (auto-generated) |
| `secret.key` | Fernet encryption key (auto-generated) |
| `requirements.txt` | Python package list |
| `recordings/` | Output folder for all video recordings |

---

## Code Architecture Overview

The application is organized into four layers that each handle a distinct responsibility.

### Encryption Layer
Handles key management and config file security. On startup `load_or_create_key()`
checks for `secret.key` and either loads the existing key or generates a new one.
Every call to `save_camera_details()` writes JSON and immediately calls
`encrypt_config()` to overwrite the file with encrypted bytes. Every call to
`load_camera_details()` calls `decrypt_config()` before any data is read.

### Data Layer
Handles reading and writing camera configuration. Camera details are stored as plain
Python dicts matching the JSON structure. The `camera_details_to_config()` function
converts a dict into an immutable `CameraConfig` dataclass that the rest of the app uses.
The `unique_file_prefix()` function guarantees no two cameras share a recording filename
prefix, even if they share a display name.

### Worker Layer (`CameraWorker`)
Each camera runs in its own `QThread`. The worker opens the RTSP stream with OpenCV,
reads frames in a loop, emits them to the GUI via Qt signals, and writes them to disk
when recording is active. If the stream drops, the worker retries automatically. A
`reconnect_requested` flag allows the GUI to trigger a reconnect without blocking the
main thread.

### GUI Layer (`CameraTile` / `MainWindow`)
`CameraTile` is a self-contained widget that owns one `CameraWorker` and displays its
output. It receives frames and status updates through Qt signals and updates the display
independently of other tiles. `MainWindow` manages the grid of tiles, handles the
add-camera dialog, routes the save-and-encrypt calls, and runs the hourly disk cleanup
timer.

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| App shows "Could not decrypt config" | `secret.key` was deleted or replaced | Delete both `secret.key` and `camera_config.json` and rerun setup |
| Camera shows "Camera Offline" | Camera unreachable or wrong credentials | Check IP and credentials, then click Reconnect |
| Recording fails to start | Disk full or bad write path | Check available disk space and folder permissions |
| No cameras appear on launch | `camera_config.json` missing or empty | Delete the file and rerun the app to redo setup |
| App closes immediately after setup | No cameras were confirmed in the wizard | Rerun the app and complete the setup wizard |

---

## Quick Reference

```bash
# Run the app (Linux)
source venv/bin/activate
python camera_dashboard.py

# Run the app (Windows)
venv\Scripts\activate
python camera_dashboard.py

# Reset everything and redo first-time setup
rm secret.key camera_config.json
python camera_dashboard.py
```

---

*Written and maintained as part of a personal technical documentation portfolio.*
