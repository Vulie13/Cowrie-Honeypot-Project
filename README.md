# Cowrie-Honeypot-Project

## Overview 

Working in a medium-sized enterprise with public-facing services ( APIs, remote access portals). We are concerned about attackers targeting the DMZ (Demilitarized Zone) servers, especially during off-hours or via brute-force attacks. We have decided to deploy Cowrie disguised as a critical production system to observe attacks that bypass or probe the perimeter.

### Objectives

- Detect lateral movement and internal reconnaissance.
- Catch insider and external threats.
- Capture real attacker tools and post-exploitation behavior.
- Act as an early warning system with false targets.


### Requirements
- Virtual Machines: Use 2 Virtual Machines connected on the same network. Cowrie VM and Penetration VM.

- Sufficient Resources: At a minimum, Each server with 2 vCPU, 2 GB of RAM, and 30 GB of storage. This is sufficient for handling a high volume of brute-force attempts, logging the activity and Pentration Attempts.

- Operating System: Ubuntu Server 22.04 LTS as the host OS for the Cowrie software and Kali Linux as the Penetration Software 


### Honeypot Setup and Configuration

#### 1. Installation and Auto-start Configuration
-	Cowrie was chosen as it's a medium-interaction SSH honeypot that can emulate a Unix system, capturing attacker behavior without exposing real systems. Installed on an Ubuntu VM using pip install cowrie, then configured as a service with systemctl enable cowrie to ensure auto-start whenever rebooted.



#### 2. Hostname Renaming and Edit the Banner 
- Configured a realistic hostnames to increase credibility and help avoid detection by attackers who look for default honeypot indicators. Changed the hostname to "svr_04" using hostnamectl set-hostname and updated /etc/hosts to match.
  
#### 3. Port Redirection (TCP 22 → 2222)
-	To avoid conflict with any existing SSH service while maintaining the appearance of a standard SSH port to external scanners. We Configured iptables rule: iptables -t nat-A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
  
#### 4. Fake Filesystem (Honeyfs) Configuration
- To provide realistic system responses and directories that attackers expect to find on a Linux system. Modified /opt/cowrie/honeyfs to include fake /etc/passwd, /etc/shadow, and user home directories with decoy files.

### 5. Fake Services Simulation
- Running services make the honeypot appear active and more attractive to attackers. I configured Cowrie to return specific responses for the command ps aux (showing fake processes) and netstat -tuln (displaying fake listening services).

#### 6. Comprehensive Logging Setup
- To capture all attacker activities for analysis and threat intelligence the below was accomplished by
-	Enabling cowrie.log for human-readable command logging
-	Configured cowrie.json for machine-readable logs
  
#### 7. Download/Upload Monitorting.
- To capture any uploads or downloads performed by the attacker. A logging setup was configured to direct any file or programs uploaded or downloaded into the folder paths /cowrie/var/lib/cowrie/downloads/ and  /cowrie/var/lib/cowrie/Uploads


Below, cowrie serive is enabled, running and configured for auto-restart

 <img width="848" height="273" alt="image" src="https://github.com/user-attachments/assets/5056eedd-fe5d-43ee-b11f-7adef5603bd0" />


### Penetration Testing Phase
1.	Host Discovery Scan using  Nmap -sn. This verifies if  the honeypot is active. nmap -sn 192.168.10.110 confirmed host availability
 
<img width="889" height="157" alt="image" src="https://github.com/user-attachments/assets/8b0957fd-0de9-48b9-a764-20698efdfbd3" />


2.	Service and OS Detection (Nmap -O -sV) .This helps identify what services the honeypot was presenting. The command nmap -O -sV 192.168.10.110 -p 2222 correctly identified SSH service on port 2222.  This is important  for understanding attack surface and verifying honeypot deception
 
 <img width="975" height="323" alt="image" src="https://github.com/user-attachments/assets/1bda9c4f-4635-4465-b646-84214baeccc6" />


3. Brute Force Attack using Hydra to test credential security and logging capabilities. Username list was ubuntu and admin. Password list: Top 1000 rockyou.txt passwords plus custom entries
   
 hydra -l ubuntu -P rockyou.txt 192.168.10.110 -shh  -s 2222 -V
 
- l/L is for target username or text file with a list of users. -p is a path to the password text file. Next is Target protocol (ssh) and IP address (192.168.10.110). -V is verbose mode which shows login attempts in real time.

  <img width="876" height="662" alt="image" src="https://github.com/user-attachments/assets/c8743afa-1397-4c4f-a22b-7ca849e82962" />

 
3.	Successful SSH Access
Ssh connection using the username attached with the IP address of the  host and -p being the port number (2222)discovered from nmap. Accessed the host using the credentials provied by hydra (user:ubuntu & pwd:IlovePenTests). Successful access lead to navigating to directories for explotation. ssh ubuntu@192.168.10.110 -p 2222

 
<img width="975" height="643" alt="image" src="https://github.com/user-attachments/assets/cc7df888-e93f-4b73-860c-bb45a468aa1a" />


4.	Explore Cowrie Honeypot
Ran ps aux and netstat -tuln commands successfully ran on honeypot for make it realist and active

<img width="975" height="285" alt="image" src="https://github.com/user-attachments/assets/a6a85fd4-c569-4a03-8c4c-c53e024d6550" />

netstat -tuln commands

<img width="975" height="246" alt="image" src="https://github.com/user-attachments/assets/4c326cc6-3649-4223-be86-56f5ecb8ab0b" />

 
Navigate into directories

 <img width="975" height="643" alt="image" src="https://github.com/user-attachments/assets/d9b5e1ed-992b-475f-b71d-41a533c45e2d" />


Downloading file.txt from cowrie

<img width="910" height="352" alt="image" src="https://github.com/user-attachments/assets/69f6fa22-deda-421d-b0b5-a2c648e2910e" />

 
Uploading file onto cowrie

<img width="975" height="93" alt="image" src="https://github.com/user-attachments/assets/f23a5d59-0f52-4224-8c36-465b72484049" />

 

## HONEYPOT LOGS
The login attempts we from the hacker were recorded (cowrie.log) and below shows that the access into the honeypot was via username ubuntu and pwd IlovePenTests.
 
<img width="975" height="160" alt="image" src="https://github.com/user-attachments/assets/602cb12d-7772-4342-bd84-b8269e8896e2" />


The commands used we recorded displaying exactly what the hacker did and which files he accessed.
  
<img width="975" height="514" alt="image" src="https://github.com/user-attachments/assets/9b3195fc-1e3b-4d33-b5c3-bfcfd5af2724" />


Downloaded file from the hacker was recorded in the downloads folder. Open the file to show the kind of information the attacker got.

 <img width="975" height="506" alt="image" src="https://github.com/user-attachments/assets/452617da-0dd7-4c5c-96e5-2b29fa373f98" />




