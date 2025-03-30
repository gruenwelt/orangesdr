# Orange Pi Zero 2W + Airspy Driver & SpyServer Setup (Headless Armbian Minimal Community)

This guide walks you through setting up an Orange Pi Zero 2W to run [SpyServer](https://airspy.com/quickstart/) in **headless mode** with an **Airspy Discovery HF+** using **Armbian Minimal (Bookworm)**. It's designed for a lightweight, low-power, reliable SDR streaming setup.

---

## 🧰 Part 1: Headless Armbian Setup

### ✅ What You’ll Need

- Orange Pi Zero 2W
- Airspy Discovery HF+
- Antenna 
- microSD card (16GB or larger)  
- Card reader  
- Wi-Fi credentials  
- Power supply for Orange Pi Zero 2W (5V, 2A) 
- Another device (Mac/PC) on the same network and with and SDR software installed (e.g. SDR++) for the setup and testing

---

### 🚀 Step-by-Step Instructions

#### 1. Download and Flash Armbian

- Download the CLI version from  
  👉 https://www.armbian.com/orange-pi-zero-2w  
- Flash to SD card using [Balena Etcher](https://www.balena.io/etcher/)

---

#### 2. Create Autoconfig File for Headless Boot

Mount the **root filesystem partition** (not `/boot`) and edit the following file:

**Path:** `/root/.not_logged_in_yet`  
**Contents:**

```bash
#/root/.not_logged_in_yet

# Network
PRESET_NET_CHANGE_DEFAULTS="1"
PRESET_NET_ETHERNET_ENABLED=0
PRESET_NET_WIFI_ENABLED=1
PRESET_NET_WIFI_SSID="YOUR_WIFI_SSID"
PRESET_NET_WIFI_KEY="YOUR_WIFI_PASSWORD"
PRESET_NET_WIFI_COUNTRYCODE="DE"
PRESET_CONNECT_WIRELESS="n"
PRESET_NET_USE_STATIC="0"

# Locale & Timezone
SET_LANG_BASED_ON_LOCATION="n"
PRESET_LOCALE="en_US.UTF-8"
PRESET_TIMEZONE="Europe/Berlin"

# Root user
PRESET_ROOT_PASSWORD="orangepizero2w"

# Normal user
PRESET_USER_NAME="sdradmin"
PRESET_USER_PASSWORD="orangepizero2w"
PRESET_DEFAULT_REALNAME="Your Name"
PRESET_USER_SHELL="bash"
```
Enter your values within the ""

More setup options can be found here: https://docs.armbian.com/User-Guide_Autoconfig/

💾 Save and safely eject the SD card.

---

#### 3. Insert SD & Power the Pi

- Insert the SD card  
- Plug in power  
- Wait about 60–90 seconds for first boot

---

#### 4. Find IP with `arp -a`

From another device on the same network:

```bash
arp -a
```
  
If unsure, flush ARP and ping the network:

```bash
sudo arp -a -d
ping 192.168.0.255
arp -a
```
  
Look for a new IP with orangepizero2w host name (e.g. 192.168.0.xxx)

⸻

5. SSH In

```bash
ssh sdradmin@discovered-ip e.g. 192.168.0.xxx
```

Replace 'discovered-ip e.g. 192.168.0.xxx' with the ip of the orange pi

Password: 
```bash
orangepizero2w
```


⸻

## 🧰 Part 2: Airspy HF+ Discovery + SpyServer Setup

✅ Tested On:
	•	Orange Pi Zero 2W
	•	Armbian Minimal (Bookworm)
	•	Airspy HF+

⸻

🔧 Setup Instructions

1. Update & Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git cmake libusb-1.0-0-dev build-essential wget
```


⸻

2. Build & Install Airspy Discovery HF+ Driver

```bash
cd ~
git clone https://github.com/airspy/airspyhf.git
cd airspyhf
mkdir build && cd build
cmake -DLIBUSB_INCLUDE_DIR=/usr/include/libusb-1.0 ..
make
sudo make install
sudo ldconfig
```


⸻

3. Set udev Rules for USB Access

```bash
echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="03eb", ATTRS{idProduct}=="800c", MODE:="0666", GROUP="plugdev"' | sudo tee /etc/udev/rules.d/52-airspyhf.rules
```

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
sudo usermod -aG plugdev $USER
newgrp plugdev
```


⸻

4. Test the Driver

```bash
airspyhf_info
```
  
✅ You should see serial number, firmware, and supported sample rates.

⸻

5. Download & Extract SpyServer

```bash
cd ~
wget https://www.airspy.com/downloads/spyserver-arm64.tgz
tar -xvzf spyserver-arm64.tgz
```

This provides:
	•	spyserver
	•	spyserver.config (default config is sufficient)

⸻

6. Run SpyServer

```bash
./spyserver spyserver.config
```

✅ You should see output like:

Listening for connections on 0.0.0.0:5555
Connected to Airspy HF+ device



⸻

7. Connect from SDR++

On another device running SDR++:
	•	Source: SpyServer
	•	Host: 192.168.0.xxx (your Orange Pi’s IP)
	•	Port: 5555
	•	Username/password: leave blank

⸻

⛓️ 8. Auto-start SpyServer on Boot (systemd)

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/spyserver.service
```

Paste:

```bash
[Unit]
Description=SpyServer
After=network.target

[Service]
Type=simple
User=sdradmin
ExecStart=/home/sdradmin/spyserver /home/sdradmin/spyserver.config
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
  
Then enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable spyserver
sudo systemctl start spyserver
```
  
✅ SpyServer now starts automatically after every reboot.

⸻

🧼 Optional Cleanup

Uninstall Airspy Driver + SpyServer

```bash
sudo rm /usr/local/lib/libairspyhf* /usr/local/bin/airspyhf*
sudo rm /etc/udev/rules.d/52-airspyhf.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
rm -rf ~/airspyhf ~/spyserver ~/spyserver-arm64.tgz ~/spyserver.config
```
  
Remove Build Tools

```bash
sudo apt remove --purge -y git cmake libusb-1.0-0-dev build-essential
sudo apt autoremove -y
```


⸻

✅ Final Notes

You now have a compact, headless SpyServer SDR setup running on the Orange Pi Zero 2W — perfect for remote HF monitoring, low-power SDR stations, or portable deployments.
