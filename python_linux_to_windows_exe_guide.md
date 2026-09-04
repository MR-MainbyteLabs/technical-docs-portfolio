
# Python Script: Linux → Windows EXE Conversion Guide

**Document Type:** Setup Guide
**Author:** MainByte Labs
**Platform:** Windows 10/11 + Python 3.11
**Skill Level:** Beginner-friendly
**Last Updated:** September 2026

---

## What This Guide Covers

Takes a Python script you wrote or edited on Linux and packages it into a standalone
`.exe` file that runs on any Windows machine — no Python installation required on
the end user's machine.

---

## How to Use This Guide

Work through Steps 1–6 in order the first time. Once you have a successful build,
use the **Quick Rebuild Checklist** at the bottom for all future updates.

---

## Prerequisites

Before starting, confirm you have:

- A working `.py` script on Linux
- Access to a Windows 10 or 11 machine (physical or VM)
- Basic comfort with copy/paste into a terminal or Command Prompt

---

## Before You Begin

Two rules govern this entire workflow:

| Rule | Reason |
|---|---|
| EXEs must be built on Windows | Cross-compilation from Linux is not supported |
| Always use Python 3.11 | Best compatibility with PyInstaller and common libraries |

Always apply the `SCRIPT_DIR` fix in Step 1 before building — skipping it causes
config and data files to save to a temporary system folder instead of beside your EXE.

---

## Step 1 — Prepare Your Script on Linux

Before transferring your script to Windows, make two edits. These ensure the built
EXE saves config files and data to the correct folder at runtime.

### 1a — Confirm `import sys` is present

Check the top of your script. If `import sys` is not already there, add it with
your other imports.

### 1b — Fix the script directory line

PyInstaller changes how Python locates the running script. The standard `__file__`
variable points to a temporary folder when the EXE runs, which causes config and
data files to save in the wrong place.

Find this line if it exists:

```python
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
```

Replace it with:

```python
SCRIPT_DIR = os.path.dirname(sys.executable) if getattr(sys, 'frozen', False) else os.path.dirname(os.path.abspath(__file__))
```

If your script has no `SCRIPT_DIR` line at all, add this block after your imports:

```python
import sys
SCRIPT_DIR = os.path.dirname(sys.executable) if getattr(sys, 'frozen', False) else os.path.dirname(os.path.abspath(__file__))
```

> **How this works:** `getattr(sys, 'frozen', False)` returns `True` when the script
> is running as a compiled EXE, and `False` when running as a plain `.py` file.
> This single line handles both cases correctly.

---

## Step 2 — Transfer Your Script to Windows

Copy your `.py` file to the Windows machine and place it in a clean, dedicated folder:

```
C:\Projects\YOUR_APP_NAME\
```

Use a USB drive, shared folder, cloud storage, or any file transfer method available
to you. Keeping the script in its own folder prevents build output from cluttering
other directories.

---

## Step 3 — Install Python 3.11 on Windows

1. Go to: https://www.python.org/downloads/release/python-3119/
2. Scroll to the **Files** section at the bottom of the page
3. Download **Windows installer (64-bit)** — filename: `python-3.11.9-amd64.exe`

> **Note:** If the page offers a `.msix` file by default, ignore it. Scroll down to
> the Files table and download the `.exe` installer directly.

4. Run the installer and choose the appropriate PATH option:

| Your situation | Action |
|---|---|
| Other Python versions are installed | **Uncheck** "Add Python to PATH" |
| This is the only Python on this machine | **Check** "Add Python to PATH" |

Complete the installation with all other options at their defaults.

---

## Step 4 — Install Dependencies

Open **Command Prompt** and install PyInstaller using the full Python 3.11 path:

```cmd
C:\Users\YOUR_USERNAME\AppData\Local\Programs\Python\Python311\python.exe -m pip install pyinstaller
```

Then install every library your script imports. Install only what your script actually
uses — check the `import` statements at the top of your script to confirm what's
needed. For example:

```cmd
C:\Users\YOUR_USERNAME\AppData\Local\Programs\Python\Python311\python.exe -m pip install pyqt5 opencv-python requests numpy
```

**To find your exact Python 3.11 path**, run:

```cmd
where python
```

Look for the result containing `Python311` and use that full path in all commands.

---

## Step 5 — Build the EXE

Navigate to your script folder in Command Prompt:

```cmd
cd C:\Projects\YOUR_APP_NAME
```

Then run the PyInstaller build command:

```cmd
C:\Users\YOUR_USERNAME\AppData\Local\Programs\Python\Python311\python.exe -m PyInstaller --onefile --windowed --name YOUR_APP_NAME yourscript.py
```

**Flag reference:**

| Flag | Effect |
|---|---|
| `--onefile` | Packages everything into a single `.exe` file |
| `--windowed` | Suppresses the console window — use for GUI apps only |
| `--name YOUR_APP_NAME` | Sets the output filename of the EXE |
| `yourscript.py` | Replace with your actual script filename |

> **Command-line scripts:** Remove `--windowed` if your app runs in a terminal.
> Keeping it on a CLI app will suppress all output and make the EXE appear to do nothing.

---

## Step 6 — Retrieve Your EXE

PyInstaller places the finished EXE at:

```
C:\Projects\YOUR_APP_NAME\dist\YOUR_APP_NAME.exe
```

Copy the `.exe` file into its own folder wherever you intend to run or distribute it.
Config files, data folders, and any output the app generates will appear alongside
the EXE at runtime.

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `ModuleNotFoundError` when running EXE | A dependency was not installed before building | Run `pip install MODULE_NAME` then rebuild |
| Config or data files saving to wrong location | `SCRIPT_DIR` fix was not applied | Apply the fix in Step 1 and rebuild |
| `pyinstaller` command not found | PyInstaller is not on the system PATH | Use the full path: `python.exe -m PyInstaller` |
| Downloaded `.msix` instead of `.exe` installer | Default download link on Python site | Re-download from the Files table at the bottom of the release page |
| EXE crashes with no error message | `--windowed` suppresses crash output | Remove `--windowed`, rebuild, run from Command Prompt to read the error |

---

## Quick Rebuild Checklist

Use this after any update to your script:

1. Edit your `.py` file on Linux
2. Transfer it to Windows into the same project folder
3. Run the PyInstaller command from Step 5
4. Collect the updated EXE from `dist\`

---

## Quick Reference — Build Command

```cmd
C:\Users\YOUR_USERNAME\AppData\Local\Programs\Python\Python311\python.exe -m PyInstaller --onefile --windowed --name YOUR_APP_NAME yourscript.py
```

**Output location:** `C:\Projects\YOUR_APP_NAME\dist\YOUR_APP_NAME.exe`

---

*Written and maintained as part of a personal technical documentation portfolio.*
```
