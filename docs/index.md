# SOP: Change Device Name from UBIQCM4XXX to JTURPCM4XXX

## Purpose
This document explains, in simple and clear terms, how to:
- Change the device name from UBIQCM4XXX to JTURPCM4XXX
- Remove Remote.it software from the device
- Update the device software (UB_JTEDS)
- Verify SIM card and 4G internet connectivity
- Change the device hostname

This guide is written so that a person with minimal technical knowledge can follow it step by step.

---

## Prerequisites
Before starting, ensure the following:
- Laptop or PC (Windows, Linux, or macOS)
- Internet connection
- Device powered on
- Device connected to the internet (hotspot or LAN)
- Device IP address
- UB_JTEDS folder available on the PC

---

## Step 1: Connect to the Device Using SSH
SSH allows you to log in to the device remotely using a command line.

### Open Terminal or Command Prompt
- Windows: Command Prompt or PowerShell
- Linux / macOS: Terminal

### Login Command
```bash
ssh UbiqCM4@<DEVICE_IP>
```

Example:
```bash
ssh UbiqCM4@192.168.1.25
```

If prompted:
- Type yes
- Enter the device password

You are now logged into the device.

---

## Step 2: Upload New Software (UB_JTEDS)

### Backup Existing Software
Run the following command on the device:
```bash
mv UB_JTEDS UB_JTEDS_backup
```
This keeps a backup of the existing software.

---

### Copy New UB_JTEDS from PC to Device
On the PC terminal:
```bash
cd ~/Desktop
scp -r UB_JTEDS UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/
```

Example:
```bash
scp -r UB_JTEDS UbiqCM4@192.168.1.25:/home/UbiqCM4/
```

Wait until the copy process completes.

---

### Set Correct Permissions
Log in to the device and run:
```bash
cd /home/UbiqCM4
chmod -R 755 UB_JTEDS
```

---

## Step 3: Remove Remote.it Software

### Update Device Packages
```bash
sudo apt update && sudo apt upgrade -y
```

---

### Stop Remote.it Services
```bash
sudo systemctl stop remoteit
sudo systemctl stop demuxer
sudo systemctl stop schannel
```

---

### Disable Remote.it Services at Boot
```bash
sudo systemctl disable remoteit
sudo systemctl disable demuxer
sudo systemctl disable schannel
```

---

### Remove Remote.it Completely
```bash
sudo apt purge remoteit -y
sudo rm -rf /usr/share/remoteit
sudo rm -rf /etc/remoteit
```

---

### Verify Removal
```bash
ps aux | grep remoteit
```
If no output appears, Remote.it has been successfully removed.

---

## Step 4: Restart Device Services
```bash
sudo systemctl daemon-reload
sudo systemctl restart main
sudo systemctl restart system_control
sudo systemctl restart logged_data_sync
sudo systemctl restart handle_4G
```

---

### Verify Software Operation
```bash
journalctl -u main -f
```

Expected output:
```text
Connected to MQTT broker
```

---

## Step 5: Update Device Access Token

Old format:
```
UBIQCM4XXX
```

New format:
```
JTURPCM4XXX
```

Example:
```
UbiqCM4@268 → JTURPCM4268
```

---

## Step 6: Verify SIM Card and 4G Connectivity
```bash
python3 handle_4G.py
```

Expected output:
```text
VI APN configured
Internet connected
4G modem & usb0 detected
```

---

## Step 7: Change Device Hostname

### Check Current Hostname
```bash
hostnamectl
```

---

### Set New Hostname
```bash
sudo hostnamectl set-hostname JTURPCM4XXX
```

Example:
```bash
sudo hostnamectl set-hostname JTURPCM4268
```

---

### Update Hosts File
```bash
sudo nano /etc/hosts
```

Change:
```text
127.0.1.1    CM4268
```

To:
```text
127.0.1.1    JTURPCM4268
```

Save and exit:
- CTRL + O, then Enter
- CTRL + X

---

### Reboot the Device
```bash
sudo reboot
```

Wait approximately two minutes, then power the device off and on again.

---

### Verify Final Hostname
```bash
hostnamectl
```

The hostname should now display:
```text
JTURPCM4XXX
```

---

## Optional: Copy Code from Device to PC
```bash
scp -r UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/UB_JTEDS ~/Desktop/
```

---

## Completion
The device is now:
- Renamed to JTURPCM4XXX
- Free of Remote.it software
- Running updated UB_JTEDS software
- Verified for SIM and 4G connectivity
- Ready for deployment
