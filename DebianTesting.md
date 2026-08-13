# Post-Installation Configuration Guide

## 1. Repository Architecture (DEB822) and Notice Cleanup

Modern Debian releases structure package sources using .sources files inside /etc/apt/sources.list.d/. To avoid redundancy and warnings, the legacy /etc/apt/sources.list file must remain completely blank.

### Main Repository Configuration

Open the central sources file with administrative privileges:  
```bash
sudo nano /etc/apt/sources.list.d/debian.sources
```

Configure the contents using the DEB822 format below. This ensures a continuous *rolling-release* workflow targeting the *Testing* branch, assigns explicit GPG signatures (Signed-By), and enables all required components (main contrib non-free non-free-firmware):  

```txt
# Official Debian Testing (Forky) Repositories  
Types: deb deb-src  
URIs: http://deb.debian.org/debian/  
Suites: testing testing-updates  
Components: main contrib non-free non-free-firmware  
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# Security Updates for Debian Testing  
Types: deb deb-src  
URIs: http://security.debian.org/debian-security/  
Suites: testing-security  
Components: main contrib non-free non-free-firmware  
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

### Mitigating "Missing Signed-By" Warnings & Orphaned Sources

If the installer left secondary configuration files pointing to legacy releases (such as trixie-backports) without validation keys, APT will trigger Notice: Missing Signed-By warnings.

> 1. **Locate conflicting files:** Scan the APT directory for outdated release references:  
>    grep \-rn "trixie-backports" /etc/apt/

> 2. **Remove orphaned source files:** If a secondary file (e.g., debian-backports.sources) is found, remove it safely:  
>    sudo rm /etc/apt/sources.list.d/debian-backports.sources

> 3. **Clear the legacy file:** To empty the traditional sources.list without permission issues caused by shell redirection (\>), edit it directly:  
>    sudo nano /etc/apt/sources.list

>    *(Delete any residual lines manually, save with Ctrl \+ O, and exit with Ctrl \+ X).*

Sync the package database and apply the full system upgrade:  
sudo apt update && sudo apt full-upgrade

⚠️ **GOLDEN RULE:** Always use sudo apt full-upgrade on Debian Testing. As a rolling release, this command properly resolves continuous dependency changes, package additions, and removals without breaking system integrity.

### **Bug Mitigation Tools**

Protect your installation against severe package regressions by installing tools that query Debian's bug tracking system prior to upgrades:  

```bash
sudo apt install apt-listbugs apt-listchanges
```

## 2. Native Hardware Graphics Acceleration (Radeon \- VA-API)

```bash
sudo apt install firmware-amd-graphics mesa-va-drivers libegl-mesa0 libgl1-mesa-dri
```

### **Multimedia Diagnostics**

Install diagnostic tools to audit OpenGL rendering and verify hardware acceleration for modern codecs (H.264, HEVC, VP9, and AV1):  

```bash
sudo apt install clinfo lm-sensors mesa-utils vainfo && vainfo && glxinfo | grep "OpenGL renderer"
```

## 3. Accelerometer Sensor Support

Enable support for touchscreens, 360-degree hinges, and automatic display orientation in tablet mode by installing the accelerometer daemon:  

```bash
sudo apt install iio-sensor-proxy
```

ℹ️ **Note:** The service activates automatically. Simply log out or reboot for KDE Plasma to detect the sensor and display rotation settings under **System Settings \> Display & Monitor**.

## 4. PipeWire

Establish a low-latency, modern audio server with full support for Bluetooth codecs by deploying PipeWire and WirePlumber:  

```bash
sudo apt install pipewire-audio wireplumber pipewire-pulse pipewire-alsa pipewire-jack
```

Enable and start the user-space services persistently:  

```bash
systemctl \--user enable \--now pipewire.service pipewire-pulse.service wireplumber.service
```

## 5\. Flatpak

```bash
sudo apt install flatpak plasma-discover-backend-flatpak && flatpak remote-add \--if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## 6. Preventive Backup Strategy and Maintenance

Create system restoration snapshots prior to running major system upgrades using Timeshift (RSYNC mode):  
sudo apt install timeshift

Keep your local disk clean by purging obsolete downloaded .deb packages and safely removing orphaned dependencies:  
sudo apt autoclean && sudo apt autoremove

## 7. AMD GPU Overclock

### Enable Overclocking Features

Open the bootloader configuration file

```bash
sudo nano /etc/default/grub
```

Unlock driver-level OverDrive functionality

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.ppfeaturemask=0xffffffff"
```

Apply settings to the boot configuration

```bash
sudo update-grub
sudo reboot
```

Verify feature mask status

```bash
cat /sys/module/amdgpu/parameters/ppfeaturemask
```

### Install LACT (Linux AMD Control Tool)

Download the latest .deb package directly from the official LACT GitHub release page

```bash
wget https://github.com/ilya-zlobintsev/LACT/releases/latest/download/lact_amd64.deb
```

Install the package using apt

```bash
sudo apt install ./lact_amd64.deb
rm lact_amd64.deb
```

Enable and start the background daemon

```bash
sudo systemctl enable --now lactd
```

## 8. ROCm

Install the ROCm components and add your user to the required groups:
```bash
sudo apt update && sudo apt install rocminfo rocm-smi libamdhip64-dev rocm-opencl-icd -y && sudo usermod -aG video,render $USER
```

Verify the installation using the SMI tool:
```bash
rocm-smi
```

## 9. Docker

Add the official Docker repository and GPG key:
```txt
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: trixie
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Install the Docker engine and related plugins:
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Note

After installation, verify that the Docker service is running:

```bash
sudo systemctl status docker
```

If Docker is not running, start it manually:

```bash
sudo systemctl start docker
```

## 10. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

## 11. VSCode

```bash
sudo apt install wget gpg &&
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /usr/share/keyrings/microsoft.gpg
```

```txt
sudo tee /etc/apt/sources.list.d/vscode.sources <<EOF
Types: deb
URIs: https://packages.microsoft.com/repos/code
Suites: stable
Components: main
Architectures: amd64,arm64,armhf
Signed-By: /usr/share/keyrings/microsoft.gpg
EOF

sudo apt update &&
sudo apt install code
```

## 12. Steam (with tweaks)

Enable Multi-Arch (32-bit support):

```bash
sudo dpkg --add-architecture i386
sudo apt update
```

```bash
sudo apt install steam-installer
```

### Gamemode

```bash
sudo apt install gamemode
```

Setup in Steam:
1. Open Steam and go to your Library.
2. Right-click your game and select Properties.
3. In the General tab, find Launch Options and input:

```txt
gamemoderun %command%
```

### Mangohud

```bash
sudo apt install mangohud goverlay
```

Configure MangoHud via GOverlay:
- Launch GOverlay from your application menu (or run goverlay in the terminal).
- Ensure the MangoHud tab is selected on the left panel.
- Toggle and customize your desired metrics (FPS counter, CPU/GPU temperature, RAM usage, positioning, fonts, frame rate limiters, etc.).
- Click Save in the bottom-left corner to instantly apply and generate your configuration file.

Setup in Steam:

```bash
gamemoderun mangohud %command%
```

### ProtonGE

```bash
flatpak install flathub net.davidotek.pupgui2
```

Launch ProtonUp-Qt from your system application menu.

Ensure Steam is selected in the installation target dropdown, click Add version, pick GE-Proton (latest version recommended), and click Install.

Restart Steam. Right-click your game > Properties > Compatibility, check Force the use of a specific Steam Play compatibility tool, and select your newly installed GE-Proton version.

### NTSYNC

Load the Kernel Module

```bash
sudo modprobe ntsync
```
Enable ntsync automatically at boot:

```bash
echo "ntsync" | sudo tee /etc/modules-load.d/ntsync.conf
```
Check that the driver loaded and created the character device node:

```bash
lsmod | grep ntsync
ls -l /dev/ntsync
```
*You should see output indicating ntsync is loaded and /dev/ntsync exists with crw-rw-rw- permissions.*

### Env vars

```bash
cat << 'EOF' > ~/.config/environment.d/proton.conf
# --- Native Wayland & HDR ---
PROTON_ENABLE_WAYLAND=1
PROTON_ENABLE_HDR=1
DXVK_HDR=1
ENABLE_HDR_WSI=1

# --- FSR & Upscaling ---
WINE_FULLSCREEN_FSR=1
WINE_FULLSCREEN_FSR_STRENGTH=2
PROTON_FSR4_UPGRADE=1

# --- Synchronization ---
PROTON_USE_NTSYNC=1
EOF
```
**Note on Compatibility:** *Some of these features may have issues with some games, you can override the global setting for that specific game*

```bash
PROTON_ENABLE_WAYLAND=0 %command%
```

## 13. Audio DSP (EQ)

Install EasyEffects, a powerful GUI-based audio processor for real-time equalization, compression, and audio enhancement. This works seamlessly with PipeWire to provide professional-grade DSP capabilities:

```bash
sudo apt update && sudo apt install easyeffects calf-plugins lsp-plugins
```

### Verify Installation

Check that PipeWire audio server is active and EasyEffects can detect audio devices:

```bash
pactl info && which easyeffects
```
