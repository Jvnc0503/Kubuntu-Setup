# **Post-Installation Configuration Guide**

**Target Environment:** AMD Ryzen / Radeon 780M | KDE Plasma (Wayland) | 2-in-1 Convertible Laptops  
This guide provides explicit, practical instructions for configuring and maintaining a clean **Debian Testing (Forky/Sid)** environment following a fresh installation. It is tailored to maximize performance on **AMD Ryzen** processors with **Radeon 780M** integrated graphics running **KDE Plasma (Wayland)**, keeping the system free of obsolete packages and configuration cruft.

## **1\. Repository Architecture (DEB822) and Notice Cleanup**

Modern Debian releases structure package sources using .sources files inside /etc/apt/sources.list.d/. To avoid redundancy and warnings, the legacy /etc/apt/sources.list file must remain completely blank.

### **Main Repository Configuration**

Open the central sources file with administrative privileges:  
sudo nano /etc/apt/sources.list.d/debian.sources

Configure the contents using the DEB822 format below. This ensures a continuous *rolling-release* workflow targeting the *Testing* branch, assigns explicit GPG signatures (Signed-By), and enables all required components (main contrib non-free non-free-firmware):  
\# Official Debian Testing (Forky) Repositories  
Types: deb deb-src  
URIs: http://deb.debian.org/debian/  
Suites: testing testing-updates  
Components: main contrib non-free non-free-firmware  
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

\# Security Updates for Debian Testing  
Types: deb deb-src  
URIs: http://security.debian.org/debian-security/  
Suites: testing-security  
Components: main contrib non-free non-free-firmware  
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

### **Mitigating "Missing Signed-By" Warnings & Orphaned Sources**

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
sudo apt install apt-listbugs apt-listchanges

## **2\. Native Hardware Graphics Acceleration (AMD Radeon 780M \- VA-API)**

To leverage the RDNA3 architecture of the Radeon 780M iGPU and offload video decoding/encoding from the CPU under Wayland, install the official AMD graphics firmware and native VA-API Mesa drivers (bypassing deprecated standards like VDPAU):  
sudo apt install firmware-amd-graphics mesa-va-drivers libegl-mesa0 libgl1-mesa-dri

### **Multimedia Diagnostics**

Install diagnostic tools to audit OpenGL rendering and verify hardware acceleration for modern codecs (H.264, HEVC, VP9, and AV1):  
sudo apt install clinfo lm-sensors mesa-utils vainfo && vainfo && glxinfo | grep "OpenGL renderer"

## **3\. Sensor Support for Convertible Laptops (2-in-1)**

Enable support for touchscreens, 360-degree hinges, and automatic display orientation in tablet mode by installing the accelerometer daemon:  
sudo apt install iio-sensor-proxy

ℹ️ **Note:** The service activates automatically. Simply log out or reboot for KDE Plasma to detect the sensor and display rotation settings under **System Settings \> Display & Monitor**.

## **4\. Advanced Multimedia Stack with PipeWire**

Establish a low-latency, modern audio server with full support for Bluetooth codecs by deploying PipeWire and WirePlumber:  
sudo apt install pipewire-audio wireplumber pipewire-pulse pipewire-alsa pipewire-jack

Enable and start the user-space services persistently:  
systemctl \--user enable \--now pipewire.service pipewire-pulse.service wireplumber.service

## **5\. Isolated Application Support via Flatpak**

Install proprietary or third-party applications (such as Steam or Discord) in sandbox environments without modifying base system libraries, integrated directly into KDE Discover:  
sudo apt install flatpak plasma-discover-backend-flatpak && flatpak remote-add \--if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

## **6\. Preventive Backup Strategy and Maintenance**

Create system restoration snapshots prior to running major system upgrades using Timeshift (RSYNC mode):  
sudo apt install timeshift

Keep your local disk clean by purging obsolete downloaded .deb packages and safely removing orphaned dependencies:  
sudo apt autoclean && sudo apt autoremove  
