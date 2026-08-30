# Linux Headless Camera System — Network Setup & Remote Access Guide

**Document Type:** Technical Setup Guide  
**Platform:** Linux (Ubuntu/Debian-based)  
**Skill Level:** Beginner to Intermediate  
**Last Updated:** 2026

---

## What This Guide Covers

This guide documents the setup of a two-machine Linux system where:

- A **headless target machine** (no monitor) runs a camera dashboard automatically on boot
- A **host machine** connects remotely to view and control it

Remote access is handled two ways:
- **SSH** — terminal access for maintenance and file transfer
- **VNC** — graphical desktop access for viewing the camera feed and managing the system

By the end of this guide the system will:

- Automatically start the camera dashboard on the target after every boot
- Stream the camera desktop to the host automatically
- Provide a separate maintenance desktop for admin access
- Allow secure file transfers between machines

---

## System Overview

| Component | Role |
|---|---|
| Target (Side-Station) | Headless machine running the camera system |
| Host (Fight) | Main workstation used for remote access |
| VNC Port 5900 | Maintenance desktop — manual admin access |
| VNC Port 5901 | Camera desktop — automatic connection from host |
| SSH | Terminal access and file transfer |

---

## Phase 1 — Network Setup

### Set a Static IP Address

Assigning a static IP ensures the target machine is always reachable at the same address, even after a reboot.

Set the target's IP within your local network range (this guide uses `192.168.1.0/24`).

### Scan the Network

Use these commands to inspect your current network and identify connected devices:

```bash
ip -br addr          # Show network interfaces and their IP addresses
ip route             # Show routing table
ip neigh             # Show ARP table (recently seen devices)
```

**Install nmap for deeper network scanning:**

```bash
sudo apt install nmap
nmap -sn 192.168.1.0/24    # Scan all devices on the subnet
arp -a                      # Show cached ARP entries
```

---

## Phase 2 — SSH Setup

SSH provides secure terminal access to the target machine from the host.

### Install and Enable SSH on the Target

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

### Allow SSH Through the Firewall

Replace `192.168.1.X` with the host machine's actual IP address:

```bash
sudo ufw allow from 192.168.1.X
```

### Connect From the Host

```bash
ssh USERNAME@192.168.1.X
```

> When prompted, type `yes` to accept and store the SSH key. You will only be asked once per machine.

---

## Phase 3 — VNC Setup (Two-Desktop System)

This system uses two separate VNC services running on the target simultaneously:

| Service | Display | Port | Purpose |
|---|---|---|---|
| x11vnc | `:0` | 5900 | Maintenance desktop — always available |
| TigerVNC | `:1` | 5901 | Camera desktop — auto-started on boot |

Having two desktops keeps the camera feed isolated from admin activity.

---

### 3A — Install Xfce Desktop Environment

The headless target needs a lightweight desktop environment. Xfce is used here for its low resource usage.

```bash
sudo apt install xfce4 xfce4-panel xfwm4 xfdesktop4
```

> At the login screen, select the **Xfce session** — not the default session.

---

### 3B — x11vnc Service (Maintenance Desktop — Port 5900)

x11vnc shares the target's existing display (`:0`) over the network.

**Install x11vnc:**

```bash
sudo apt install x11vnc
```

**Create the systemd service:**

```bash
sudo nano /etc/systemd/system/x11vnc.service
```

**Paste the following:**

```ini
[Unit]
Description=x11vnc server for headless desktop
After=display-manager.service
Requires=display-manager.service

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc \
  -display :0 \
  -auth /var/run/lightdm/root/:0 \
  -forever \
  -shared \
  -loop \
  -noxdamage \
  -repeat \
  -rfbport 5900

Restart=always
RestartSec=3

[Install]
WantedBy=graphical.target
```

**Enable and start the service:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
sudo systemctl status x11vnc.service
```

---

### 3C — TigerVNC Service (Camera Desktop — Port 5901)

TigerVNC creates a separate virtual desktop (`:1`) dedicated to the camera dashboard.

**Install TigerVNC:**

```bash
sudo apt install tigervnc-standalone-server tigervnc-common
```

**Create a VNC password:**

```bash
vncpasswd
```

**Create the desktop startup file:**

```bash
nano ~/.vnc/xstartup
```

**Paste the following:**

```bash
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4
```

**Make it executable:**

```bash
chmod +x ~/.vnc/xstartup
```

**Create the TigerVNC systemd service:**

```bash
sudo nano /etc/systemd/system/tigervnc-camera.service
```

Paste your project-specific service definition here.

**Enable and start the service:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable tigervnc-camera.service
sudo systemctl start tigervnc-camera.service
sudo systemctl status tigervnc-camera.service
```

**Verify the port is listening:**

```bash
ss -tlnp | grep 5901
```

---

## Phase 4 — Autostart Camera Dashboard on Boot

The camera dashboard Python script needs to launch automatically when the camera desktop starts.

**Create the autostart directory:**

```bash
mkdir -p ~/.config/autostart
```

**Create the autostart entry:**

```bash
nano ~/.config/autostart/camera-dashboard.desktop
```

**Paste the following:**

```ini
[Desktop Entry]
Type=Application
Name=Camera Dashboard
Exec=/home/side/Python/venv/bin/python /home/side/Python/Cam_System/camera_dashboard.py
Terminal=false
Hidden=false
X-GNOME-Autostart-enabled=true
X-XFCE-Autostart-enabled=true
```

**Verify the dashboard is running:**

```bash
ps aux | grep camera_dashboard
```

---

## Phase 5 — Host Auto-Connect to Camera Desktop

The host machine automatically connects to the camera desktop (port 5901) on startup using a user-level systemd service.

**Create the service file:**

```bash
nano ~/.config/systemd/user/cam-vncviewer.service
```

**Paste the following:**

```ini
[Unit]
Description=Auto-start Camera VNC Viewer
After=graphical-session.target

[Service]
Type=simple
ExecStart=/usr/bin/vncviewer 192.168.1.105:5901
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

**Enable and start the service:**

```bash
systemctl --user daemon-reload
systemctl --user enable cam-vncviewer.service
systemctl --user start cam-vncviewer.service
systemctl --user status cam-vncviewer.service
```

---

## Phase 6 — File Transfer with SCP

SCP (Secure Copy Protocol) transfers files between machines over SSH.

### Basic Syntax

```bash
scp [options] SOURCE DESTINATION
```

Either the source or destination can be a remote machine.

### Common Transfer Patterns

**Download a file from target to host:**

```bash
scp USERNAME@192.168.1.105:/path/to/file.mp4 .
```

> The `.` means "place the file in the current directory."

**Download a file to a specific location on the host:**

```bash
scp USERNAME@192.168.1.105:/path/to/file.mp4 /home/fight/Downloads/
```

**Upload a file from host to target:**

```bash
scp /home/fight/file.txt USERNAME@192.168.1.105:/home/side/
```

**Copy an entire directory:**

```bash
scp -r USERNAME@192.168.1.105:/home/side/Documents/Project .
```

### If You Don't Know the File Path

Find the file on the target first:

```bash
find / -name "filename.mp4" 2>/dev/null
```

Then copy using the returned path:

```bash
scp USERNAME@192.168.1.105:/home/side/path/to/filename.mp4 /home/fight/Downloads/
```

### Sending Files From the Target Back to the Host

If you are already SSH'd into the target and want to push a file to the host:

```bash
scp /path/to/file.mp4 fight@192.168.1.101:/home/fight/Downloads/
```

---

## Phase 7 — Router Access (OpenWrt)

| Method | Command / URL |
|---|---|
| Web UI (LuCI) | `http://192.168.1.1` |
| SSH (standard) | `ssh root@192.168.1.1` |
| SSH (older firmware) | `ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa root@192.168.1.1` |

> Default user: `root`

---

## System Verification

Run these checks after setup or after a reboot to confirm everything is working correctly.

**On the target:**

```bash
systemctl status x11vnc.service           # Maintenance VNC running?
systemctl status tigervnc-camera.service  # Camera VNC running?
ss -tlnp | grep -E "5900|5901"            # Both ports listening?
ps aux | grep camera_dashboard            # Dashboard process running?
```

**On the host:**

```bash
systemctl --user status cam-vncviewer.service    # Auto-connect running?
```

### Expected Results

| Check | Expected State |
|---|---|
| Port 5900 | Maintenance desktop available |
| Port 5901 | Camera desktop available |
| Camera Dashboard | Running automatically after boot |
| Host VNC viewer | Connected automatically to port 5901 |

---

## Manual Maintenance Access

The maintenance desktop is always available regardless of the camera system state.

**Connect manually from the host:**

```bash
vncviewer 192.168.1.105:5900
```

**Install TigerVNC viewer on the host if needed:**

```bash
sudo apt install tigervnc-viewer
```

---

## Quick Reference

```bash
# SSH into target
ssh USERNAME@192.168.1.105

# Manual VNC — maintenance desktop
vncviewer 192.168.1.105:5900

# Manual VNC — camera desktop
vncviewer 192.168.1.105:5901

# Check all services
systemctl status x11vnc.service
systemctl status tigervnc-camera.service
systemctl --user status cam-vncviewer.service

# Transfer a file from target to host
scp USERNAME@192.168.1.105:/path/to/file .

# Find a file on the target
find / -name "filename" 2>/dev/null
```

---

*Written and maintained as part of a personal technical documentation portfolio.*
