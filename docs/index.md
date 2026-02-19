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

## Step 3 : Correct your main.service(Most Important to get the data in JT Portal)

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




## Step 4: Remove Remote.it Software

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

## Step 5: Restart Device Services
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

## Step 6: Update Device Access Token


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

## Step 7: Verify SIM Card and 4G Connectivity
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

## Step 8: Change Device Hostname

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
127.0.1.1    JTURPCM4XXX
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


## 9. Install Tailscale (Remote Access)

Download the Statis Binary file named : arm64: tailscale_1.94.2_arm64.tgz (This should be on download)



Copy file to device:

```bash
scp tailscale_1.94.2_arm64.tgz UbiqCM4@<DEVICE_IP>:~
```


Extract the tgz file and navigate to that folder :

```bash
tar -xvzf tailscale_1.94.2_arm64.tgz
cd tailscale_1.94.2_arm64
```

Install:

```bash

sudo cp tailscale /usr/local/bin/
sudo cp tailscaled /usr/local/sbin/
sudo chmod 755 /usr/local/bin/tailscale
sudo chmod 755 /usr/local/sbin/tailscaled
sudo cp tailscaled /usr/local/sbin/
sudo chmod 755 /usr/local/sbin/tailscaled
sudo chmod 755 /usr/local/bin/tailscale
```
Check the version to tailscale this will ensure that tailscale is installed properly:

```bash
tailscale version
```
Create service:

```bash
sudo nano /etc/systemd/system/tailscaled.service
```

Paste:

```ini
[Unit]
Description=Tailscale node agent
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/sbin/tailscaled
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo mkdir -p /var/lib/tailscale
sudo systemctl daemon-reload
sudo systemctl enable tailscaled
sudo systemctl daemon-reload
sudo systemctl reset-failed
sudo systemctl restart tailscaled

```


Check the status of Tailscaled:

```bash
systemctl status tailscaled
```

It should show Runnning.


Start the tainet channel:
```bash
sudo tailscale up

```
Approve device in browser.

Get remote IP:

```bash
tailscale ip -4
```

Remote SSH from anywhere:

```bash
ssh UbiqCM4@100.x.x.x
```

---





## Optional: Copy Code from Device to PC
```bash
scp -r UbiqCM4@<DEVICE_IP>:/home/UbiqCM4/UB_JTEDS ~/Desktop/
```

---

## Final Technician Checklist (MANDATORY)

- Write the device **Access Token on device body using permanent marker**
- Label the SIM card as **JT**
- Note the **Tailscale IP in deployment sheet**
- Verify remote SSH login works
- Confirm data visible in JT portal

---

## Deployment Complete
Device is now ready for field installation and remote maintenance.

