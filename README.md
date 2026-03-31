# How to Install Wi-Fi Driver for the BrosTrend AX900 WiFi & Bluetooth Adapter (Realtek RTL8851BU) on Ubuntu

This guide covers the installation of the `rtw89` driver for the [BrosTrend AX900 WiFi Bluetooth USB Adapter](https://www.brostrend.com/products/wb1?srsltid=AfmBOoqYnU2w1SEVmSyPXpVXHX-QbRrMNiAHWpwthUokaG8oGgzLkN4N) (Hardware ID `0bda:b851`) on Ubuntu Linux. 

**Note:** You might confuse this chipset with the RTL8852BU. If your `lsusb` shows `0bda:b851`, you *must* use the `rtw89` unified driver, not the older standalone 8852 drivers.

## Prerequisites
Before starting, ensure your system has an active internet connection (via Ethernet, Bluetooth tethering, or USB tethering from a smartphone) to download the necessary files.

### 1. Disable Secure Boot
If your computer has Secure Boot enabled in the BIOS/UEFI, it will block unsigned third-party drivers from loading. 
* Check your status by running: `mokutil --sb-state`
* If enabled, reboot into your BIOS settings and disable Secure Boot before proceeding.

---

## Installation Steps

### Step 1: Install Build Dependencies
You need the Linux headers that match your current kernel, as well as the standard compiling tools. Open your terminal and run:

```bash
sudo apt update
sudo apt install -y build-essential bc dkms linux-headers-$(uname -r) git
```

### Step 2: Download the Driver Source Code
We will use the highly recommended community repository maintained by `morrownr`, which includes the unified `rtw89` driver stack.

```bash
cd ~
git clone https://github.com/morrownr/rtw89.git
```

### Step 3: Compile the Driver
Navigate into the newly downloaded folder and compile the code.

```bash
cd ~/rtw89
make clean
make
```
*(Wait for the compilation to finish. Ensure there are no fatal errors at the end of the output).*

### Step 4: Install the Driver
Install the compiled modules into your system's kernel directory.

```bash
sudo make install
```

### Step 5: Update the Kernel Module Index
Force the kernel to refresh its list of available drivers so it recognizes the newly installed files.

```bash
sudo depmod -a
```

### Step 6: Load the Module
The installation script appends `_git` to the driver name to prevent conflicts with default, incomplete kernel drivers. Load the correct module by running:

```bash
sudo modprobe rtw89_8851bu_git
```

### Step 7: Verify the Installation
Unplug your USB Wi-Fi adapter, wait 3 seconds, and plug it back in. Run the following command to check if the network interface was created:

```bash
ip link
```
You should see a new interface starting with `wlx` followed by a MAC address (e.g., `wlx7419f817483b`). 

### Step 8: Connect to Wi-Fi
You can now connect to your Wi-Fi network using the standard Ubuntu GUI network manager in the top right corner of your screen, or via the terminal using `nmcli`.

---

## Troubleshooting

**"No such device" or Driver Fails to Bind**
If you previously installed an incorrect driver (like `rtl8852bu`), it may be conflicting with the new one. Ensure you remove old drivers by navigating to their original installation folder and running their respective `sudo ./remove-driver.sh` scripts before installing `rtw89`.

**Module Not Found Error**
If `modprobe` returns a fatal error saying the module is not found, verify that the `make` command in Step 3 completed successfully without missing dependencies. Check the directory manually:
```bash
find /lib/modules/$(uname -r) -type f -name "*.ko*" | grep rtw89_8851bu
```
