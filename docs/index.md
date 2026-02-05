# SOP: Change Device Name from UBIQCM4XXX to JTURPCM4XXX

## Purpose
This document explains, in simple and clear terms, how to:
- Change the device name from UBIQCM4XXX to JTURPCM4XXX
- Remove Remoteit software from the device
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


```bash
UbiqCM4
```

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

In case of Linux
```bash
cd ~/Desktop
scp -r UB_JTEDS UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/
```

In case of Windows

```bash
scp -r "Add UB_JTEDS file location followed by file name " UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/
```
Example : 
```bash
scp -r "C:\Users\Admin\OneDrive\Desktop\UB_JTEDS" UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/
```
Refer the screenshot for more information
![SCP in Windows](assets/images/scp_windows.jpeg)


Wait until the copy process completes.

---

### Set Correct Permissions
Log in to the device and run:
```bash
cd /home/UbiqCM4
chmod -R 755 UB_JTEDS
```
This will have no output
---

## Step 4 : Correct your main.service(Most Important to get the data in JT Portal)

### Edit the service file

```bash
sudo systemctl edit --full main

```


### Replace the [Service] section with THIS

```bash
[Unit]
Description=Main Python Script Service
After=network.target pigpiod.service
Requires=pigpiod.service

[Service]
Type=simple
User=UbiqCM4

WorkingDirectory=/home/UbiqCM4/UB_JTEDS
ExecStart=/usr/bin/python3 /home/UbiqCM4/UB_JTEDS/main.py

Restart=on-failure
RestartSec=5

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target


```

### Apply changes (DON’T SKIP)

```bash
sudo systemctl daemon-reload
sudo systemctl restart main


```

### Check

```bash
systemctl status main
journalctl -u main -f



```




## Step 3: Remove Remote.it Software

### Update Device Packages

```bash
sudo apt update && sudo apt upgrade -y
```
This might take some time 

Check for running Remoteit services

```bash
ps aux | grep remoteit
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
If no output appears or the output is something like the screenshot below , Remoteit has been successfully removed.


![SCP in Windows](assets/images/Remoteit.jpeg)

---

## Step 4: Restart Device Services
```bash
sudo systemctl daemon-reload
sudo systemctl restart main
sudo systemctl restart system_control
sudo systemctl restart logged_data_sync
sudo systemctl restart handle_4G
```
This will have no output.
---

### Verify Software Operation
```bash
journalctl -u main -f
```

Expected output:
```text
Connected to MQTT broker and No sensor output.
```

---

## Step 5: Update Device Access Token


Navigate inside UB_JTEDS :

```bash
cd UB_JTEDS
```
Open the access_token.json file :

```bash
sudo nano access_token.json
```

Old format:
```
UbiqCM4@XXX
```

New format:
```
JturpCM4@XXX
```
Note: Please write the access token on the box with marker.

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


### If that doesn't work then follow this
```bash
python3 fetch_sim_info.py
```

Expected output:
```text
SIM info
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

### Reboot the CM4
```bash
sudo reboot
```

### Reverify Hostname
```bash
hostnamectl
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
This will restart the system from software and hardware both
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
- Please contact the developer to add the device in JT portal.
