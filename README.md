# Reconnaissance Machine
## Overview
This project demonstrates how to revive old systems for security researching tasks;
I transformed an abandoned computer into a dedicated reconnaissance machine for target server enumeration and automated attacks.
By taking advantage of an already existing old, unused hardware, I was able to cut the monthly/yearly expenses of owning a VPS (Virtual Private Server) by building a more flexible, scalable, and local VS (Virtual Server)
that operates as a backbone for my home-lab security research.

## Objectives
- Turning my old computer from a dust-collecting pile of metal into a cheeky packet-firing machine.
- Install and boot automatically into  Kali Linux alongside Windows OS.
- Automate the logging process in Kali Linux.
- Start SSH & HTTP services on boot.

## Main Goal
As soon as the computer powers up, the entire process—from booting into Kali Linux to starting the necessary services—runs automatically, with no human interaction required.

## Hurdles & Complications
- Dual Boot failed (Windows & Kali Linux).
- Live Boot—from a flash drive—failed.
- WSL (Windows Subsystem for Linux) failed.

### Failure Causes, respectively
- Unable to detect DHCP server on network configuration process upon installing Kali Linux.
- I don't have a flash drive with a large storage (and I don't wanna buy one :P).
- Can't switch NAT to Bridged mode persistently on each reboot.

## The Sole Solution
Download Virtual Box and install Kali Linux image in it.

## Getting Hands Dirty
### Downlaoding Virtual Box & Installing Kali Linux Image
Come on, man! It's way too easy.
> [!CAUTION]
> This write-up was published on February 10th, 2025. The following links might be—100%—outdated, seek the up-tp-date versions from the official channels.
1) [Download Virtual Box for Windows](https://www.virtualbox.org/wiki/Downloads)
2) [Download Kali Linux Image for VB](https://cdimage.kali.org/kali-2024.4/kali-linux-2024.4-virtualbox-amd64.7z)
3) Decompress Kali Linux Image & double-click on the orange box to install the image on VB.

### Enabling Auto-Login
LightDM allows users to bypass the login screen, automatically logging into a predefined user account without requiring them to enter a password.
<br>
No-human interaction. Perfect for my case.
<br> <br>
Write in the terminal:
```
sudo vim /etc/lightdm/lightdm.conf
```
and change the following entries:
```
[Seat:*]
autologin-user=your_kali_username
autologin-user-timeout=0
```
![lightdm](https://github.com/user-attachments/assets/e7c58925-0c95-495a-8d91-8c27dee249bb)

then restart the LighDM service:
```
sudo systemctl restart lightdm
```

### Enabling SSH Service
Port 22—SSH will be utilized as the only way to access the recon machine.
<br> <br>
To run SSH persistently on boot:
```
sudo systemctl enable --now ssh
```
![ssh1](https://github.com/user-attachments/assets/d53aded4-a677-4c88-8900-fc9c3dbb8b99)

### Enabling HTTP Service
This will be the recon webserver, where it will store the result files of the conducted reconnaissance & attacks activities.
<br>
I used Systemd to run the webserver persistently on boot. <br> It is a system and service manager for Linux operating systems. It is designed to bootstrap the user space and manage system processes after booting,
including starting, stopping, and managing system services (daemons) and other resources.
<br> <br>
Create a folder where the web server will be hosted. Mine is: /home/kali/bb
<br>
then make a service file inside the directory for user-defined systemd services:
```
sudo vim /etc/systemd/system/Reconserver.service
```
and write inside the service file:
```
[Unit]
Description=Simple HTTP Server
After=network.target

[Service]
ExecStart=/usr/bin/python3 -m http.server 8080
WorkingDirectory=your_hosted_folder
Restart=always
User=your_username

[Install]
WantedBy=multi-user.target
```
![http](https://github.com/user-attachments/assets/d99649df-5d64-464f-935a-3c8cba3dab07)

Lastly, enable sytemctl to run the custom service:
```
sudo systemctl enable --now Reconserver
```
> **Note:** If you added the custom service file but systemctl didn't recongize it, run:
```
sudo systemctl daemon-reload
```
This will reload systemd to recognize changes, then run *systemctl enable* again.

### Creating Task Schedule
This is the last step to conclude the steps to build a recon machine.
<br>
In this step, I will schedule a task to fire up the Kali Linux image inside the Virtual Box. To be able to do that, we must know the name of the image.
<br>
Open Virtual Box and navigate to the image you have been working on as the recon machine VS (Virtual Server)
![2](https://github.com/user-attachments/assets/8ed20a8a-e489-4b15-9a16-a7e5861618df)
Take note of the name then press *Windows Button + R* and write: taskschd.msc
![1](https://github.com/user-attachments/assets/78912320-eccb-40df-8493-6ae6a3fd7542)
<br> Press on: Create Task
![3](https://github.com/user-attachments/assets/32e1071d-d6c2-4aff-b959-55dfe0a018cb)
<br> and fill the task details as follows:

![4](https://github.com/user-attachments/assets/61131630-e435-4d60-b5c5-f8d3027b796f)
![5](https://github.com/user-attachments/assets/4f77e9c6-85a5-4814-a591-361bf01cc61b)
<br> On the Actions page, create a new action with the following command:
```
Program/Script: "location_of_your_VirtualBox_folder\Virtual Box\VBoxManage.exe"
Add arguments (optional): startvm "your_image_name"
```
![6](https://github.com/user-attachments/assets/5ff0ba7f-a4f6-45be-a15f-dd7bef4019a8)
![7](https://github.com/user-attachments/assets/95265ad4-6a42-48fd-bab9-96286196541e)
![8](https://github.com/user-attachments/assets/0e0686f0-9e88-45b8-957f-140127464ba9)

### Power & Sleep
Turn off any sleep mode from both the Windows and the Kali Linux image.
![power](https://github.com/user-attachments/assets/4c9107e1-632d-49c0-9a57-3f1e099e74a8)
![power2](https://github.com/user-attachments/assets/6b0f14af-8919-493e-b2ee-b8f6eb0a717b)
![power3](https://github.com/user-attachments/assets/bf54c31c-f85e-4701-9394-6473ee13d2fc)
![power 4](https://github.com/user-attachments/assets/bd068c90-97b1-481f-82ba-6258a20daf06)



### Checking The Lab Setup
Save everything then restart the computer, cross your legs, and see the magic unflod before your eyes...
<br> After you have booted into your Kali Linux image (aka, your new recon machine)—successfully, hopefully—do a last check by running the following command:
```
ps aux | grep -E 'ssh|http'
```
If you got this
![make sure everything works](https://github.com/user-attachments/assets/36a3a2d6-5f2d-4627-a748-64f58cea1517)
it means everything works error-free.

### Testing The Recon Machine
#### SSH
![ssh](https://github.com/user-attachments/assets/60f07492-ffb9-451c-810d-a20466403348)

#### HTTP
![test1](https://github.com/user-attachments/assets/2245bd86-7cfd-4369-ae70-1dfdd5daeada)
![website](https://github.com/user-attachments/assets/ca809368-9e5f-47e5-82c9-cb6db94dc07f)
![test2](https://github.com/user-attachments/assets/16e6b2e6-2040-4ab4-8f07-efea2950d3dc)
![test3](https://github.com/user-attachments/assets/4a66e7a1-8d7c-4baa-bd30-4e4b405c4c21)
