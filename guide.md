# Kubuntu 25.10 gaming setup

## Phase 0: System Preparation

### 1. First update
Recommended to do periodically.
```bash
sudo apt update && sudo apt upgrade
```

### 2. Block Snaps
Create an APT preference file to prevent `snapd` from being reinstalled automatically, then update package list:
```bash
# Block snapd via APT preferences
sudo tee /etc/apt/preferences.d/nosnap.pref <<EOF
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF

# Update package lists
sudo apt update
```

### 3. Install Flatpak
```bash
# Update your package index
sudo apt update

# Install the core Flatpak utility and the Discover backend integration
sudo apt install -y flatpak plasma-discover-backend-flatpak

# Add Flathub to the remote list for all users on the system
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Reboot your system to apply the new environment variables
sudo reboot
```

## Phase I: Drivers & Core Dependencies

### 1. Enable 32-bit Architecture & Install Build Tools
Steam and some older games require 32-bit compatibility libraries.
```bash
sudo dpkg --add-architecture i386
sudo apt update && sudo apt upgrade -y
sudo apt install linux-headers-$(uname -r) build-essential dkms vainfo mesa-utils -y
```

### 2. Enable NTSYNC Kernel Module
The `ntsync`module provides advanced synchronization primitives that significantly improve performance for Windows games running through Wine/Proton.
```bash
echo 'ntsync' | sudo tee /etc/modules-load.d/ntsync.conf
sudo modprobe ntsync
```

## Phase II: Applications & Utilities

### 1. Mozilla Firefox (Official APT Repository)
Because Ubuntu/Kubuntu replaces the default `firefox` APT package with a transitional package that installs the Snap version, we must explicitly add Mozilla's repository and set its priority to ensure you receive the native `.deb` package.
```bash
# 1. Create keyring directory and download the Mozilla signing key
sudo install -d -m 0755 /etc/apt/keyrings
wget -q [https://packages.mozilla.org/apt/repo-signing-key.gpg](https://packages.mozilla.org/apt/repo-signing-key.gpg) -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null

# 2. Add the Mozilla APT repository using the modern deb822 format
sudo tee /etc/apt/sources.list.d/mozilla.sources <<EOF
Types: deb
URIs: [https://packages.mozilla.org/apt](https://packages.mozilla.org/apt)
Suites: mozilla
Components: main
Signed-By: /etc/apt/keyrings/packages.mozilla.org.asc
EOF

# 3. Configure APT priority to strictly prefer the native Mozilla packages
sudo tee /etc/apt/preferences.d/mozilla <<EOF
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000
EOF

# 4. Update the package index and install native Firefox
sudo apt update
sudo apt install firefox -y
```

### 2. Steam & Gamemode
```bash
sudo apt install steam-installer gamemode -y
```

### 3. Mangohud & Goverlay
```bash
# Update your package index
sudo apt update

# Install the overlay, the GUI manager, and the testing tools
sudo apt install -y mangohud goverlay vulkan-tools mesa-utils
```

### 4. OBS Studio (Official PPA)
```bash
sudo add-apt-repository ppa:obsproject/obs-studio -y
sudo apt update
sudo apt install obs-studio -y
```

### 5. Visual Studio Code (Microsoft APT Repository)
```bash
# Update package index and install dependencies
sudo apt update
sudo apt install software-properties-common wget gpg apt-transport-https

# Import the Microsoft GPG key
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg &&
sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft.gpg &&
rm -f microsoft.gpg

# Add the VS Code repository to your system
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'

# Update package cache and install VS Code:
sudo apt update
sudo apt install code
```

### 6. Zoom

## Phase III: Gaming Setup

### GEProton

#### Install ProtonUp-Qt (Flatpak)

### Gamemode

### Mangohud

#### Configure using Goverlay

## Phase IV: KDE Plasma & System Tweaks

### Display

### Mouse

### Power Management

### Background Services