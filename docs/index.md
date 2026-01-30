
# SOP: Change UBIQCM4XXX to JTURPCM4XXX

## Overview
This document describes the **Standard Operating Procedure (SOP)** to:
- Change device ID from `UBIQCM4XXX` to `JTURPCM4XXX`
- Disable and remove Remote.it services
- Update UB_JTEDS code
- Change SIM and verify 4G
- Update hostname
---
## 🔐 Accessing the Device via SSH

This section describes the procedure to securely access the CM4 device using **SSH (Secure Shell)** from a personal computer.

This procedure is applicable to **Windows, Linux, and macOS** systems.

---

### 1. SSH Command

To log in to the device, open a **command prompt / terminal** on your PC and execute the following command:

```bash
ssh UbiqCM4@<DEVICE_IP>


---

## 1. Upload UB_JTEDS Code
Upload the folder **UB_JTEDS** written by Saikat.

This ensures the device number changes from:
```
UBIQCM4XXX → JTURPCM4XXX
```

---

## 2. Disable & Remove Remote.it Services

### Update System
```
sudo apt update && sudo apt upgrade -y
```

### Identify Services
```
ps aux | grep remoteit
```

Common services:
- demuxer  
- schannel

### Stop Services
```
sudo systemctl stop remoteit
sudo systemctl stop demuxer
sudo systemctl stop schannel
```

### Disable on Boot
```
sudo systemctl disable remoteit
sudo systemctl disable demuxer
sudo systemctl disable schannel
```

### Remove Completely
```
sudo apt purge remoteit -y
sudo rm -rf /usr/share/remoteit
sudo rm -rf /etc/remoteit
```

### Verify
```
ps aux | grep remoteit
```
No output = success.

---

## 3. Code Update (UB_JTEDS)

### Backup Existing Code
```
ssh UbiqCM4@<DEVICE_IP>
mv UB_JTEDS UB_JTEDS_old_$(date +%F_%H-%M)
```

> To find IP: Mobile Hotspot → Manage Devices

### Copy New Code (From PC)
```
cd ~/Desktop
scp -r UB_JTEDS UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/
```

### Fix Permissions
```
ssh UbiqCM4@<DEVICE_IP>
cd /home/UbiqCM4
chmod -R 755 UB_JTEDS
```

### Restart Services
```
sudo systemctl daemon-reload
sudo systemctl restart main
sudo systemctl restart system_control
sudo systemctl restart logged_data_sync
sudo systemctl restart handle_4G
```

### Verify Logs
```
journalctl -u main -f
```
Expected:
```
Connected to MQTT broker
```

---

## 4. Update Access Token

Update `access_token` inside code:
```
JTURPCM4@XXX
```
Example:
```
UbiqCM4@268 → JTURPCM4268
```

---

## 5. SIM & 4G Verification

Run:
```
python3 handle_4G.py
```

Expected Output:
```
VI APN configured
Internet connected
4G modem & usb0 detected
```

---

## 6. Change Device Hostname

### Check Current
```
hostnamectl
```

### Update Hostname
```
sudo hostnamectl set-hostname JTURPCM4XXX
```

### Update Hosts File
```
sudo nano /etc/hosts
```

Change:
```
127.0.1.1    CM4268
```
To:
```
127.0.1.1    JTURPCM4268
```

Save:
- CTRL + O → Enter  
- CTRL + X

### Reboot
```
sudo reboot
```

Wait 2 minutes → Power cycle device.

### Verify
```
hostnamectl
```

---

## 7. Useful SCP Command

Download code from device to PC:
```
scp -r UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/UB_JTEDS ~/Desktop/
```

---

## SOP Completed Successfully
