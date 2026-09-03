# Python Script: Linux → Windows EXE Conversion Guide
**Author:** MainByte Labs | **Last Updated:** September 2026
**Skill Level:** Beginner-friendly | **Platform:** Windows 10/11 + Python 3.11

---

## What This Guide Does
Takes a Python script you wrote or edited on Linux and packages it into a 
standalone `.exe` file that runs on any Windows machine — no Python install required.

---

## Prerequisites
Before starting, confirm you have:
- A working `.py` script on Linux
- Access to a Windows 10 or 11 machine (physical or VM)
- Basic comfort with copy/paste into a terminal or Command Prompt

---

## Important Rules
- EXEs **must be built on Windows** — cross-compiling from Linux does not work
- Always use **Python 3.11** — best compatibility with PyInstaller and common libraries
- Always apply the `SCRIPT_DIR` fix below — skipping it causes config/data files 
  to save to a temp folder instead of beside your EXE

---

## Step 1 — Prepare Your Script on Linux

### 1a. Confirm `import sys` is at the top of your script
If it's not there, add it.

### 1b. Fix the script directory line
Find any line like this:
```python
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
```
Replace it with:
```python
SCRIPT_DIR = os.path.dirname(sys.executable) if getattr(sys, 'frozen', False) else os.path.dirname(os.path.abspath(__file__))
```
If your script has no such line, add both of these near the top, after your imports:
```python
import sys
SCRIPT_DIR = os.path.dirname(sys.executable) if getattr(sys, 'frozen', False) else os.path.dirname(os.path.abspath(__file__))
```

---

## Step 2 — Transfer Your Script to Windows
Copy your `.py` file to the Windows machine into a clean, dedicated folder:
C:\Projects\YourAppName\
Use a USB drive, shared folder, cloud storage, or any method you prefer.

---

## Step 3 — Install Python 3.11 on Windows

1. Go to: https://www.python.org/downloads/release/python-3119/
2. Scroll to the **Files** section at the bottom
3. Download **Windows installer (64-bit)** — filename: `python-3.11.9-amd64.exe`
4. Run the installer:
   - Other Python versions already installed → **uncheck "Add Python to PATH"**
   - This is the only Python on the machine → **check "Add Python to PATH"**

---

## Step 4 — Install Dependencies
Open Command Prompt and install PyInstaller using the full Python 3.11 path:
C:\Users\YourName\AppData\Local\Programs\Python\Python311\python.exe -m pip install pyinstaller

Then install any libraries your script uses, for example:
C:\Users\YourName\AppData\Local\Programs\Python\Python311\python.exe -m pip install pyqt5 opencv-python requests numpy

> **Only install what your script actually imports.** Check the top of your script 
> for `import` statements to see what's needed.

**To find your exact Python 3.11 path**, run:
where python
Look for the line containing `Python311`.

---

## Step 5 — Build the EXE
Navigate to your script folder in Command Prompt:
cd C:\Projects\YourAppName
Then run:
C:\Users\YourName\AppData\Local\Programs\Python\Python311\python.exe -m PyInstaller --onefile --windowed --name YourAppName yourscript.py

| Flag | What it does |
|---|---|
| `--onefile` | Packages everything into a single EXE |
| `--windowed` | Removes the black console window — **remove this flag for command-line scripts** |
| `--name YourAppName` | Sets the EXE filename |
| `yourscript.py` | Replace with your actual script filename |

---

## Step 6 — Locate and Use Your EXE
Your finished EXE will be at:
C:\Projects\YourAppName\dist\YourAppName.exe

Move just the `.exe` into its own folder wherever you want to run it. Any config 
files or data folders the app creates will appear in the same folder as the EXE.

---

## Troubleshooting

| Error | Fix |
|---|---|
| `module not found` when running EXE | Install the missing module: `pip install modulename`, then rebuild |
| Config or data files saving to wrong location | Apply the `SCRIPT_DIR` fix in Step 1 |
| `pyinstaller` command not found | Use the full Python path: `python.exe -m PyInstaller` |
| Downloaded `.msix` instead of `.exe` installer | Re-download from the **Files** section at the bottom of the Python release page |
| EXE crashes with no error message | Remove `--windowed`, rebuild, run from Command Prompt to see the error output |

---

## Quick Rebuild Checklist
When you update your script and need a new EXE:
1. Edit your `.py` on Linux
2. Transfer to the same Windows folder
3. Run the PyInstaller command from Step 5
4. Grab the new EXE from `dist\`





