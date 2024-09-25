
# Installing Nexmon and Enabling Monitor Mode on Raspberry Pi 5

## The Problem
For my current research project, I need to do some WiFi sniffing. The basic idea is simple: put the WiFi card into monitor mode, then run airodump-ng. However, on the Raspberry Pi 5 with Debian 12, trying to enable monitor mode on the WiFi card often results in errors. 

Nexmon comes to the rescue! Nexmon is a C-based firmware patching framework for Broadcom/Cypress WiFi chips that enables features like monitor mode with radiotap headers and frame injection.

Below is a guide on how to install Nexmon and enable monitor mode.

## Step-by-Step Installation Guide

### Step 1: Update System and Prepare the Build Environment
1. Open a terminal and switch to root for the installation process:
   ```bash
   sudo su
   ```
2. Upgrade your Raspbian system:
   ```bash
   apt-get update && apt-get upgrade
   ```

### Step 2: Modify Configuration File
1. Edit the `/boot/firmware/config.txt` file to use the correct kernel:
   ```bash
   nano /boot/firmware/config.txt
   ```
2. Add the following line at the bottom:
   ```
   kernel=kernel8.img
   ```
3. Reboot your system:
   ```bash
   reboot
   ```

### Step 3: Install Dependencies
1. Install the required packages for building Nexmon:
   ```bash
   sudo apt install raspberrypi-kernel-headers git libgmp3-dev gawk qpdf bison flex make autoconf libtool texinfo xxd
   ```
2. Set up ARM architecture and install the necessary libraries:
   ```bash
   sudo dpkg --add-architecture armhf
   sudo apt-get update
   sudo apt-get install libc6:armhf libisl23:armhf libmpfr6:armhf libmpc3:armhf libstdc++6:armhf
   ```

### Step 4: Handle Symbolic Links (Check Versions)
1. Verify your installed versions of the libraries (`libisl` and `libmpfr`), and then create the required symbolic links. For example:
   ```bash
   sudo ln -s /usr/lib/arm-linux-gnueabihf/libisl.so.23.0.0 /usr/lib/arm-linux-gnueabihf/libisl.so.10
   sudo ln -s /usr/lib/arm-linux-gnueabihf/libmpfr.so.6.1.0 /usr/lib/arm-linux-gnueabihf/libmpfr.so.4
   ```

### Step 5: Clone Nexmon Repository
1. Clone the Nexmon project repository from GitHub:
   ```bash
   git clone https://github.com/seemoo-lab/nexmon.git
   ```
2. Navigate into the Nexmon root directory:
   ```bash
   cd nexmon
   ```

### Step 6: Set Up Build Environment
1. Source the environment setup script:
   ```bash
   source setup_env.sh
   ```

### Step 7: Compile Nexmon Tools and Firmware
1. Compile the build tools and extract the necessary firmware patches:
   ```bash
   make
   ```

2. Navigate to the patch directory for the `bcm43455c0` chipset (which applies to the Raspberry Pi 5):
   ```bash
   cd patches/bcm43455c0/7_45_206/nexmon
   ```

3. Compile the patched firmware:
   ```bash
   make
   ```

4. Backup your original firmware:
   ```bash
   make backup-firmware
   ```

5. Install the patched firmware:
   ```bash
   sudo -E make install-firmware
   ```

### Step 8: Install Nexutil and Enable Monitor Mode
1. Go back to the Nexmon root directory:
   ```bash
   cd ../../../../
   ```

2. Navigate to the `nexutil` folder and install it:
   ```bash
   cd nexutil
   make && sudo make install
   ```

3. To enable monitor mode on your WiFi interface (e.g., `wlan0`):
   ```bash
   nexutil -m2
   ```

### Step 9: Verify Monitor Mode
1. Use `iwconfig` or `iw` to check if monitor mode is enabled:
   ```bash
   iwconfig wlan0
   ```

   The output should confirm the interface is in monitor mode.

### Step 10: Use Airodump-ng
1. Now that monitor mode is enabled, you can run `airodump-ng`:
   ```bash
   sudo airodump-ng wlan0
   ```

---

This guide should help you successfully enable monitor mode on the Raspberry Pi 5 for WiFi sniffing using Nexmon. If you encounter issues with symbolic links or library versions, ensure they are compatible with the system architecture.
