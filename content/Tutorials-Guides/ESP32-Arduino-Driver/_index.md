---
title: ESP32 Arduino Core Installation Guide
date: 2026-03-31
description: "Installing the ESP32 Arduino core — two methods, driver setup, and the pitfalls that break a first flash."
keywords: ["ESP32 Arduino core", "ESP32 driver install", "embedded setup guide", "ESP32 tutorial"]

---

---

# Comprehensive ESP32 Arduino Core Installation Guide

## Overview: Two Installation Methods

| Method | Best For | Pros | Cons |
|--------|----------|------|------|
| **Boards Manager** | Beginners, stability | One-click install, version management, automatic updates | Slight delay for newest features |
| **Manual/Git Clone** | Developers, beta testers | Latest commits, instant updates, offline capable | Manual updates, potential instability |

---

## Method 1: Boards Manager (Recommended for Most Users)

### Universal Prerequisites
- **Arduino IDE 1.8.19+** or **Arduino IDE 2.3+** (IDE 2.x recommended)
- **Python 3.7+** (usually bundled with IDE 2.x)
- **USB Cable**: Must be data-capable, not charge-only

### Step 1: Add ESP32 Board Manager URL

**Arduino IDE 2.x:**
1. File → Preferences (or Ctrl+,)
2. In "Additional boards manager URLs" add:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Click OK

**Arduino IDE 1.8.x:**
1. File → Preferences
2. Same URL as above
3. If multiple URLs, separate with commas

### Step 2: Install the Core

1. Tools → Board → Boards Manager...
2. Search for **"ESP32"** by **Espressif Systems**
3. Select version (latest stable recommended, e.g., 3.0.x or 2.0.x)
4. Click **Install** (downloads ~500MB-1GB of toolchains)

### Step 3: Select Your Board

Tools → Board → ESP32 Arduino → Select your specific board:
- **ESP32 Dev Module** (generic, works for most ESP32-WROOM boards)
- **ESP32-S2/S3/C3 Dev Module** (for specific variants)
- **ESP32-CAM**, **TTGO T-Display**, etc. (for specific hardware)

---

## Method 2: Manual Installation (Git Clone)

Use this for development versions, unreleased fixes, or offline environments.

### Windows (PowerShell/Command Prompt)

```powershell
# 1. Create directory structure
mkdir %USERPROFILE%\Documents\Arduino\hardware\espressif
cd %USERPROFILE%\Documents\Arduino\hardware\espressif

# 2. Clone repository (shallow clone for faster download)
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32

# 3. Optional: Checkout specific version
git checkout 3.0.7  # or 2.0.17 for legacy

# 4. Download tools (requires Python)
cd tools
python get.py

# 5. Install Python dependencies (if get.py fails)
pip install pyserial
```

**Windows-Specific Notes:**
- If `python` command not found, use `py` or `python3`
- Path might be `C:\Users\<Username>\Arduino\...` if Documents folder is redirected
- Windows Defender may flag toolchain downloads as false positives—allow them

### macOS (Terminal)

```bash
# 1. Install prerequisites (if not present)
brew install python git  # or use macOS system python3

# 2. Create directories
mkdir -p ~/Documents/Arduino/hardware/espressif
cd ~/Documents/Arduino/hardware/espressif

# 3. Clone with submodules
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32

# 4. Checkout stable (optional)
git checkout 3.0.7

# 5. Get tools
cd tools
python3 get.py
```

**macOS-Specific Notes:**
- **Apple Silicon (M1/M2/M3)**: Toolchains are natively supported since ESP32 core 2.0.3+
- If you get "permission denied" on serial port: `sudo chmod 666 /dev/tty.usbserial-*`
- macOS 10.15+ requires allowing "System Software from developer Espressif" in Security & Privacy during first upload

### Linux (Universal)

#### Ubuntu/Debian/Linux Mint

```bash
# 1. User permissions (dialout group)
sudo usermod -aG dialout $USER
# Log out and back in (or reboot) after this

# 2. Install dependencies
sudo apt-get update
sudo apt-get install git python3-pip python3-pyserial python3-venv

# 3. Create directories
mkdir -p ~/Arduino/hardware/espressif
cd ~/Arduino/hardware/espressif

# 4. Clone and setup
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32
git checkout 3.0.7  # optional stable version
cd tools
python3 get.py
```

#### Fedora/RHEL/CentOS/Rocky Linux

```bash
# 1. Groups (dialout on Fedora)
sudo usermod -aG dialout $USER

# 2. Dependencies
sudo dnf install git python3-pip pyserial

# 3. Setup directories and clone (same as Ubuntu)
mkdir -p ~/Arduino/hardware/espressif
cd ~/Arduino/hardware/espressif
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32/tools
python3 get.py
```

#### Arch Linux / Manjaro

```bash
# 1. Serial permissions (uucp/lock/tty, NOT dialout)
sudo usermod -aG uucp,lock,tty $USER
# CRITICAL: Log out and back in after this step

# 2. Install dependencies
sudo pacman -S git python-pip python-pyserial

# 3. Clone ESP32 core
mkdir -p ~/Arduino/hardware/espressif
cd ~/Arduino/hardware/espressif
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32
git checkout 3.0.7  # optional: specific release
cd tools
python get.py
```

#### openSUSE

```bash
# 1. Groups
sudo usermod -aG dialout $USER

# 2. Dependencies
sudo zypper install git python3-pip python3-pyserial

# 3. Clone (same structure as above)
mkdir -p ~/Arduino/hardware/espressif
cd ~/Arduino/hardware/espressif
git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
cd esp32/tools
python3 get.py
```

---

## Platform-Specific Deep Configuration

### Windows: Driver Installation

Most ESP32 boards use **CP2102** or **CH340** USB-to-Serial chips:

**CP2102 (Silicon Labs):**
1. Download from: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers
2. Install **CP210x Universal Windows Driver**
3. Device Manager should show: "Silicon Labs CP210x USB to UART Bridge (COMx)"

**CH340/CH341 (WCH):**
1. Windows 10/11 usually auto-installs
2. If not: https://www.wch.cn/downloads/CH341SER_ZIP.html
3. Device Manager: "USB-SERIAL CH340 (COMx)"

**Verification:**
```powershell
# In PowerShell
Get-PnpDevice -Class Ports | Where-Object {$_.FriendlyName -like "*USB*"}
```

### macOS: Serial Port Permissions

Even after installation, you may get "Failed to connect to ESP32: Timed out waiting for packet header":

```bash
# Fix permissions (run each session or make permanent)
sudo chmod 666 /dev/tty.usbserial-*
# OR for specific port
sudo chmod 666 /dev/cu.usbserial-0001

# Permanent fix: Create udev-like rule (macOS uses devfs)
# Edit /etc/devfs.rules (create if doesn't exist)
sudo nano /etc/devfs.rules
# Add:
# [system=10]
# add path 'tty.usbserial*' mode 0666
# add path 'cu.usbserial*' mode 0666
```

### Linux: UDEV Rules (Permanent Fix)

Create persistent permissions so you don't need `sudo`:

```bash
# Create udev rule
sudo nano /etc/udev/rules.d/50-esp32.rules
```

**Add these lines:**
```
# CP2102 (Espressif devkits)
SUBSYSTEM=="usb", ATTR{idVendor}=="10c4", ATTR{idProduct}=="ea60", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"

# CH340/CH341
SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="7523", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"

# FT232 (some third-party boards)
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6001", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"

# ESP32-S2/S3 USB OTG (CDC)
SUBSYSTEM=="usb", ATTR{idVendor}=="303a", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
```

**Reload rules:**
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Verification & First Upload

### 1. Select Correct Settings

**Tools Menu:**
- **Board**: ESP32 Dev Module (or your specific model)
- **Port**: 
  - Windows: `COM3` (or whatever appears when board is plugged in)
  - macOS: `/dev/cu.usbserial-0001` or `/dev/tty.usbserial-*`
  - Linux: `/dev/ttyUSB0` or `/dev/ttyACM0`
- **Upload Speed**: 921600 (or 115200 if unreliable)
- **CPU Frequency**: 240MHz (WiFi/BT)
- **Flash Frequency**: 80MHz
- **Flash Mode**: QIO
- **Partition Scheme**: Default 4MB with spiffs (or Huge APP if large sketches)

### 2. Test Sketch

```cpp
// Built-in example: File → Examples → 01.Basics → Blink
// Modify LED_BUILTIN to 2 (most ESP32 devkits use GPIO2 for onboard LED)

#define LED_BUILTIN 2

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("LED ON");
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("LED OFF");
  delay(1000);
}
```

### 3. Upload Process

1. **Hold BOOT button** on ESP32 (if your board has one)
2. Click Upload in IDE
3. **Release BOOT button** when you see "Connecting..." in the console
4. If successful: "Hash of data verified" followed by "Hard resetting via RTS pin..."

**Auto-reset issues:** If upload fails consistently, you may need a 10µF capacitor between EN (RST) and GND, or use specific reset methods (varies by board design).

---

## Troubleshooting Matrix

| Symptom | Windows | macOS | Linux |
|---------|---------|-------|-------|
| **Port not visible** | Install drivers (CP2102/CH340) | Install drivers via `brew` or manually | Add user to `dialout`/`uucp` group |
| **Permission denied** | Run IDE as Administrator (temporary) | `chmod 666 /dev/tty.*` | Udev rules or `sudo usermod -aG dialout $USER` |
| **Connection timed out** | Wrong COM port, try lower baud (115200) | Check cable, try `/dev/cu.*` instead of `/dev/tty.*` | Check `dmesg \| grep tty` for port name |
| **Failed to write to target RAM** | USB 3.0 issue, try USB 2.0 hub | USB-C to USB-A adapter issue | Add user to `lock` group (Arch) |
| **Missing Python/serial** | `pip install pyserial` | `pip3 install pyserial` | `sudo pacman -S python-pyserial` (Arch) or `apt install python3-serial` |

### Specific Error Messages

**"A fatal error occurred: Failed to connect to ESP32: Timed out..."**
- **Cause**: ESP32 not in bootloader mode
- **Fix**: Hold BOOT button, click Upload, release when "Connecting..." appears
- **Permanent Fix**: Some boards need DTR/RTS capacitors soldered for auto-reset

**"Permission denied: '/dev/ttyUSB0'"**
- **Linux**: `sudo usermod -aG dialout $USER` then **reboot** (not just logout)
- **Arch**: Use `uucp` group instead of `dialout`

**"pyserial or esptool not found"**
```bash
# All platforms
pip3 install --user pyserial esptool
# Or for system-wide (Linux)
sudo pip3 install pyserial esptool
```

**"Cannot open /dev/ttyUSB0: Device or resource busy"**
- **Cause**: ModemManager is using the port
- **Fix**: `sudo systemctl stop ModemManager` (Linux) or add udev rule with `ENV{ID_MM_DEVICE_IGNORE}="1"`

---

## Advanced Topics

### Version Management (Manual Method)

```bash
cd ~/Arduino/hardware/espressif/esp32

# List available versions
git tag | grep -E "^3\.|^2\." | tail -20

# Switch to specific version
git checkout 3.0.7
git submodule update --init --recursive  # Critical step
cd tools && python get.py

# Return to bleeding edge
git checkout master
git pull
git submodule update --init --recursive
python get.py
```

### Offline Installation

For air-gapped systems:
1. On connected machine: Clone repo, run `get.py`
2. Copy entire `~/Arduino/hardware/espressif/esp32` folder to offline machine
3. Install Arduino IDE on offline machine
4. Paste folder to same location

### Multiple Arduino Versions

If using IDE 1.8 and 2.x simultaneously:
- **IDE 1.8**: Uses `~/Arduino/hardware/` (Linux/mac) or `Documents\Arduino\hardware\` (Windows)
- **IDE 2.x**: Uses `~/.arduino15/packages/` for Boards Manager installs
- Manual method works for both if placed in correct location

### ESP32-S2/S3/C3 Specifics

**USB-OTG Native (S2/S3):**
- No drivers needed on Windows 10/11, macOS, Linux
- Port appears as USB JTAG/serial debug unit
- Hold BOOT, press RST (on board), release BOOT to enter bootloader

**C3 (RISC-V):**
- Uses different toolchain (RISC-V instead of Xtensa)
- Automatically handled by core 2.0.0+

---

## Maintenance

### Updating (Boards Manager)
Tools → Board → Boards Manager → Updates (checkmark icon) → Update ESP32

### Updating (Manual)
```bash
cd ~/Arduino/hardware/espressif/esp32
git pull
git submodule update --init --recursive
cd tools
python get.py
```

### Complete Removal
**Boards Manager:** Boards Manager → ESP32 → Remove
**Manual:** `rm -rf ~/Arduino/hardware/espressif/esp32`

---

**Final Checklist:**
- [ ] Arduino IDE installed and running
- [ ] Board URL added (for Boards Manager) OR Git repo cloned
- [ ] User added to correct serial group (Linux) OR Drivers installed (Windows)
- [ ] Board selected matching your hardware
- [ ] Correct port selected
- [ ] USB cable is data-capable (test with phone file transfer)
- [ ] Bootloader mode activated (if auto-reset fails)

This guide covers 95% of installation scenarios. For specific board variants (LOLIN, M5Stack, etc.), consult the manufacturer's additional instructions after completing this base installation.

---
---

## FAQ: Why & When to Choose Manual Installation

### 1. Why should I install the ESP32 core manually instead of using Boards Manager?

**Boards Manager** downloads ~800MB–1.2GB of toolchain binaries (xtensa compilers, esptool, precompiled libraries) through Arduino IDE's package manager. This process is **atomic** — if it fails at 99%, it restarts from zero.

**Manual installation** gives you:
- **Resumable downloads**: `git clone` supports resume (`git fetch --depth=1`), and `get.py` can be re-run until completion
- ** granular control**: Download toolchains once, copy to multiple machines
- **Offline capability**: Works on air-gapped systems after initial setup
- **Bleeding-edge fixes**: Access commits newer than the latest "stable" release (e.g., urgent WiFi patches not yet in 3.0.7)

**Choose Manual if**: Your connection drops frequently, you manage multiple dev machines, or you need unreleased bug fixes.

---

### 2. I have a weak internet connection and keep getting "Download timeout" or "CRC check failed" errors. How does manual installation solve this?

**The Problem**: Arduino Boards Manager uses Java's network stack with fixed 30–60 second timeouts. On slow connections (<512kbps) or high-latency networks (satellite, rural DSL), the toolchain download (particularly the 500MB+ GCC compiler package) times out repeatedly.

**The Manual Solution**:
```bash
# Step 1: Shallow clone (only ~150MB of metadata, not full history)
git clone --depth=1 --recursive https://github.com/espressif/arduino-esp32.git esp32

# Step 2: Run get.py with resume capability
cd esp32/tools
python get.py
# If it fails at 45%, simply run python get.py again — it resumes partial files
```

**Pro tip for unstable connections**: Use `wget` or a download manager for the toolchains manually, then place them in `esp32/tools/dist/` before running `get.py` (which will detect existing files and skip re-downloading).

---

### 3. Can I resume an interrupted installation, or do I have to start over?

**Boards Manager**: No. If the progress bar freezes at 70% and you cancel, the partial cache is discarded. Next attempt restarts from 0%.

**Manual Method**: Yes, inherently resumable:
- **Git**: `git fetch --unshallow` or `git submodule update --init` resumes interrupted clones
- **Toolchains**: `get.py` checks SHA256 hashes of files in `tools/dist/`. Completed files are skipped; incomplete files are re-downloaded.

**Recovery procedure after disconnect**:
```bash
cd ~/Arduino/hardware/espressif/esp32
git submodule update --init --recursive --progress  # Shows % complete
cd tools && python get.py --verbose  # Shows which specific tool is being fetched
```

---

### 4. How do I install the ESP32 core on a computer with no internet connection (air-gapped)?

**Boards Manager**: Impossible. It requires live JSON index validation and real-time package download.

**Manual Method**: The "Sneakernet" approach:
1. **On connected machine**:
   ```bash
   mkdir -p ~/Arduino/hardware/espressif
   cd ~/Arduino/hardware/espressif
   git clone --recursive https://github.com/espressif/arduino-esp32.git esp32
   cd esp32/tools && python get.py
   cd ../../..
   tar -czf esp32-offline.tar.gz espressif/  # Compress entire core + toolchains
   ```
2. **Transfer** `esp32-offline.tar.gz` via USB drive to offline machine
3. **On offline machine**:
   ```bash
   mkdir -p ~/Arduino/hardware/
   tar -xzf esp32-offline.tar.gz -C ~/Arduino/hardware/
   ```
4. **Install Python dependencies** manually (download `.whl` files from PyPI on connected machine, transfer via USB):
   ```bash
   pip install --no-index --find-links=/path/to/usb/pyserial pyserial
   ```

**Result**: Full ESP32 support without the offline machine ever touching the internet.

---

### 5. The Boards Manager download is huge (1GB+) and fails repeatedly on my metered connection. Is manual installation smaller?

**Size comparison**:
- **Boards Manager**: Downloads *all* available toolchains (xtensa-esp32, xtensa-esp32s2, xtensa-esp32s3, riscv32-esp32c3, esp32ulp) plus all precompiled libraries — ~1.2GB
- **Manual (selective)**: You can manually download *only* the toolchain for your specific chip

**Bandwidth-saving manual approach**:
```bash
# After cloning, edit esp32/tools/get.py to comment out unneeded platforms
# Or manually download only your chip's toolchain from:
# https://github.com/espressif/crosstool-NG/releases
# Place in tools/dist/ then run get.py (it will use local files)
```

**Typical savings**: If you only use ESP32-WROOM (Xtensa), you can skip RISC-V and ULP tools, reducing download to ~400MB.

---

### 6. Does manual installation make it easier to manage multiple developer machines?

**Yes — significantly**.

**The "Golden Master" Method**:
1. Install manually on Machine A (with good internet)
2. Verify it compiles and uploads correctly
3. Archive the entire `~/Arduino/hardware/espressif/esp32` folder
4. Distribute to Machines B, C, D via local network or USB drives

**No per-machine downloads**: Each developer doesn't need to download 1GB through the company firewall.

**Version consistency**: All machines use the exact same git commit (e.g., `3.0.7`), preventing "works on my machine" issues caused by Boards Manager auto-updating on one laptop but not another.

---

### 7. Why does `get.py` fail with "Connection timeout" even when I can browse websites fine?

**Root causes**:
- **Parallel downloads**: `get.py` attempts to download 4–6 large toolchain archives simultaneously. On weak connections, this saturates the pipe and causes timeouts.
- **CDN issues**: GitHub releases (where toolchains are hosted) throttle connections from certain regions or ISPs.
- **SSL inspection**: Corporate firewalls intercept HTTPS, breaking Python's SSL verification.

**Fixes for weak connections**:
```bash
# Single-threaded download (slower but stable)
python get.py --platform esp32  # Only download ESP32 tools, not S2/S3/C3

# If behind corporate proxy
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
python get.py

# Disable SSL verify (last resort, security risk)
python get.py --ssl-no-verify
```

---

### 8. Is manual installation more stable than Boards Manager for daily development?

**Stability factors**:

| Aspect | Boards Manager | Manual/Git |
|--------|----------------|------------|
| **Update surprise** | Auto-updates can break sketches overnight | You control when to `git pull` |
| **Network dependency** | Requires internet to verify packages every launch | Fully offline after setup |
| **Corruption recovery** | Delete entire package, re-download 1GB | `git status` shows exactly which file is corrupted |
| **Rollback** | Difficult (must uninstall, reinstall old version) | `git checkout 2.0.17` instant rollback |

**Trade-off**: Manual requires you to remember to update (`git pull`), while Boards Manager auto-updates. For weak internet, manual is more stable because it won't randomly fail to compile because it can't reach the package index server.

---

### 9. How do I update the ESP32 core manually when I have limited monthly bandwidth?

**Incremental updates** (Manual advantage):
```bash
cd ~/Arduino/hardware/espressif/esp32

# Check what's new without downloading
git fetch origin
git log HEAD..origin/master --oneline --stat

# If only small changes (e.g., 3 files changed, +50 lines):
git pull origin master
# Toolchains usually don't change between minor commits, so get.py does nothing

# If major release (e.g., 3.0.7 → 3.1.0):
# Only then run get.py to fetch new toolchains if necessary
python tools/get.py
```

**Bandwidth conservation**: Git pulls are ~KBs of text. Boards Manager re-downloads entire toolchain packages even for tiny core updates.

**Selective updates**: You can update only the `libraries/` folder for bug fixes without touching the massive `tools/` folder.

---

### 10. Can I mix manual and Boards Manager installations on the same computer?

**Yes, but with caution**.

**Priority rules**:
- Arduino IDE prioritizes **Manual** (`~/Arduino/hardware/`) over Boards Manager (`~/.arduino15/packages/`)
- If both exist, the manual version is used for compilation

**Use case — "Hybrid Mode"**:
- **Install stable via Boards Manager**: For production work (reliable, no internet needed after install)
- **Install dev version manually**: For testing new features
- **Switch between them**:
  - Rename `~/Arduino/hardware/espressif` to `~/Arduino/hardware/espressif_dev` when you want Boards Manager stable
  - Rename back when you need bleeding edge

**Conflict warning**: Never have the same version in both locations (e.g., 3.0.7 in Boards Manager and 3.0.7 in manual). This causes "Multiple libraries found" warnings and undefined behavior.

**Best practice**: Use Boards Manager for stable work, manual only for specific development/testing — or uninstall Boards Manager version entirely to save disk space once manual is working.

---

### Quick Decision Matrix

| Your Situation | Recommended Method |
|----------------|-------------------|
| **DSL/Satellite/Spotty 4G** | **Manual** — resume capability essential |
| **Corporate firewall/proxy** | **Manual** — easier to configure proxy settings |
| **Single machine, good internet** | Boards Manager — simpler updates |
| **Air-gapped/Offline lab** | **Manual** — only viable option |
| **Teaching/classroom (30 students)** | **Manual** — install once, copy to 30 USB drives |
| **Need yesterday's bugfix** | **Manual** — access commits immediately, no wait for release |

**Bottom line**: Manual installation transforms the ESP32 core from a "streaming service" (requires constant reliable internet) into a "downloaded application" (works offline, survives interruptions).

---