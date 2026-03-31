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
```bash
# 1. Download the latest Zoom Debian package directly from their official server
wget https://zoom.us/client/latest/zoom_amd64.deb

# 2. Install the package using APT to handle dependencies natively
sudo apt install -y ./zoom_amd64.deb

# 3. Clean up the downloaded installation file to keep your directory tidy
rm zoom_amd64.deb
```

## Phase III: Gaming Setup

### 1. GEProton

#### Install ProtonUp-Qt (Flatpak)
```bash
# Install ProtonUp-Qt from Flathub
flatpak install -y flathub net.davidotek.pupgui2
```

(Note: If the application does not immediately appear in your KDE Plasma application menu, you can launch it manually from the terminal the first time by running `flatpak run net.davidotek.pupgui2`.)

#### Install GE-Proton using the GUI
ProtonUp-Qt provides a straightforward visual interface to handle the downloading and extraction process we previously did via the terminal.

1. Open ProtonUp-Qt from your application launcher.
2. In the main window, ensure the Install for: dropdown menu at the top is set to Steam.
3. Click the Add Version button located at the bottom of the window.
4. In the new popup window:
    - Compatibility Tool: Select GE-Proton.
    - Version: The latest stable release will be selected by default (e.g., GE-Proton-10).
5. Click Install.

#### Activation on Steam
Once the installation inside ProtonUp-Qt is completely finished, you must apply the new tool inside Steam.

1. Fully restart the Steam client. Steam only scans for new compatibility tools during its initial startup.
2. Open your Steam Library.
3. Right-click on the specific game you want to run with GE-Proton and select Properties.
4. Navigate to the Compatibility tab.
5. Check the box that says Force the use of a specific Steam Play compatibility tool.
6. Click the dropdown menu and select the GE-Proton version you just installed.

### 2. Global Environment Variables
Use systemd's `/etc/environment.d/` directory to set variables system-wide. This applies them early, before apps start, so KDE Plasma, Steam, and Proton all inherit the same settings.

```bash
sudo tee /etc/environment.d/gaming-config.conf <<EOF
# Enforce Wayland for Desktop Apps
ELECTRON_OZONE_PLATFORM_HINT=wayland
MOZ_ENABLE_WAYLAND=1

# Enable NTSYNC
WINE_USE_NTSYNC=1
PROTON_USE_NTSYNC=1

# Enable HDR support
PROTON_ENABLE_HDR=1

# Upscaling Upgrade (Choose according to your hardware)
PROTON_FSR4_UPGRADE=1
# PROTON_DLSS_UPGRADE=1
# PROTON_XESS_UPGRADE=1
EOF
```

### 3. Gamemode
To enable gamemode for your Steam games, you need to add a launch option to each game:

**For Individual Games:**
1. Open your Steam Library
2. Right-click on the game you want to optimize
3. Select **Properties**
4. In the **Launch Options** field, add:
   ```
   gamemoderun %command%
   ```
5. Close the properties window and launch the game

### 4. Mangohud

#### Configure using GOverlay
GOverlay provides a graphical interface to customize MangoHud without needing to manually edit configuration files.

1. Open **GOverlay** from your application launcher.
2. Navigate to the **MangoHud** tab.
3. Use the visual interface to toggle the metrics you want to monitor during gameplay (e.g., FPS, Frame Timing, CPU/GPU Temperature, RAM usage).
4. Click the **Save** button at the bottom right to generate your configuration profile.

#### Testing
Before launching a game, you can verify the overlay works using the testing tools we installed in Phase II.
```bash
# Test MangoHud using a basic Vulkan application
mangohud vkcube
```

#### Activation
To enable the overlay in your games, you must add it to the Steam Launch Options. If you are already using Gamemode (configured in Step 3), you can chain these commands together seamlessly.

1. Right-click the game in your Steam Library and select Properties.
2. In the Launch Options field, chain the commands separated by a space:
    ```bash
    mangohud gamemoderun %command%
    ```

## Phase IV: KDE Plasma & System Tweaks

### Display

### Mouse

### Power Management

### Background Services