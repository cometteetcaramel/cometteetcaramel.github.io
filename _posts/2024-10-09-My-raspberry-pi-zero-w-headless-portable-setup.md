
# Raspberry Pi Zero W Portable Headless Setup

In this post, we will explore how to set up a headless Raspberry Pi Zero W for various portable projects. This setup can be used for a range of projects, such as wardriving and more.

---

## Pi Setup

### Parts Required

You will need the following components to follow this tutorial:

- **Raspberry Pi Zero W** or **Raspberry Pi Zero 2 W** – Make sure to choose the W version as it includes WiFi and Bluetooth (other Pi models may work as well).
- **SD card** – An 8GB card will work, but I recommend using a 16GB card.
- **USB cable** – To power up your Pi.
- **Power bank** or **UPS hat** – To power your headless setup. You could also power it through your computer or phone with a USB adapter.

### Installing and Configuring the OS on Raspberry Pi

The first step is to install an operating system (OS) for your Pi. You have several options as long as the OS is Debian-based. For my headless setup, I prefer the lightweight version of Raspberry Pi OS (formerly known as "Raspbian").

Since I'm using Windows, I'll be using the **Raspberry Pi Imager** tool. If you're on Linux, you can install it with the command:

```bash
sudo apt install rpi-imager
```

#### Flashing the OS on the SD Card

1. Launch the Raspberry Pi Imager utility.
2. Choose your device; in this case, we will select **Raspberry Pi Zero 2 W**.
3. Choose the OS: go to **Raspberry Pi OS (Other)** -> **Raspberry Pi OS Lite (64-bit)**. If you're unsure, use the 32-bit version (though the Zero 2 W supports 64-bit).
4. Insert your SD card into the computer and choose it under **Storage**.
5. Enable **OS customisation settings** to configure the setup. Make sure to enable **SSH**.
6. Write the OS to the SD card.

#### Connecting to the Pi via SSH

Once the OS is written to the SD card:

1. Insert the SD card into the Pi and power it on.
2. Find the Pi’s IP address (you can connect a screen; the IP is shown on the boot banner).
3. Connect to the Pi via SSH:

```bash
ssh <your_user>@<your_ip>
```

#### Updating Your Pi

Once connected, update your Pi by running the following commands:

```bash
sudo apt-get update
```

```bash
sudo apt-get full-upgrade
```

Reboot the Pi to apply the updates:

```bash
sudo reboot
```

### Connect via SSH Using Bluetooth

To connect via Bluetooth, follow these steps. (Use `CTRL+X` to exit the Nano editor.)

1. Install the required Bluetooth tools:

```bash
sudo apt install bluez-tools
```

2. Create the following configuration files:

```bash
sudo nano /etc/systemd/network/pan0.netdev
```

Add the following content:

```ini
[NetDev]
Name=pan0
Kind=bridge
```

```bash
sudo nano /etc/systemd/network/pan0.network
```

Add the following content:

```ini
[Match]
Name=pan0
[Network]
Address=172.20.1.1/24
DHCPServer=yes
```

```bash
sudo nano /etc/systemd/system/bt-agent.service
```

Add the following content:

```ini
[Unit]
Description=Bluetooth Auth Agent
[Service]
ExecStart=/usr/bin/bt-agent -c NoInputNoOutput
Type=simple
[Install]
WantedBy=multi-user.target
```

```bash
sudo nano /etc/systemd/system/bt-network.service
```

Add the following content:

```ini
[Unit]
Description=Bluetooth NEP PAN
After=pan0.network
[Service]
ExecStart=/usr/bin/bt-network -s nap pan0
Type=simple
[Install]
WantedBy=multi-user.target
```

3. Enable Bluetooth on boot:

```bash
sudo systemctl enable systemd-networkd
sudo systemctl enable bt-agent
sudo systemctl enable bt-network
sudo systemctl start systemd-networkd
sudo systemctl start bt-agent
sudo systemctl start bt-network
```

4. To pair the Pi with your phone or computer, run:

```bash
sudo bt-adapter --set Discoverable 1
```

Enable Bluetooth on your device and pair it with the Pi. Once paired, enable **Use for Internet access**.

Next, SSH into the Pi using Bluetooth:

```bash
ssh <your_user>@172.20.1.1
```

### Installing Additional Tools

You can now install any additional tools you need for your portable, headless Pi setup!
