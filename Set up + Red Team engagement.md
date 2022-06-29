# Set up & Red Team Engagement: Summary of Operations

## Table of Contents
- Filebeat Setup
- Metricbeat Setup
- Packetbeat Setup
- Capstone vulnerable VM exploit

### Filebeat Setup

Ran the following commands as root user on Capstone VM:
``` bash
  filebeat modules enable apache
  filebeat setup
```
  ![](Images/filebeat%20setup.png)
  

### Metricbeat Setup

Ran the following commands as root user on Capstone VM:
 ``` bash
  metricbeat modules enable apache
  metricbeat setup
```
  ![](Images/Metricbeat%20setup.png)

### Packetbeat Setup
Ran the following command as root user on Capstone VM:
``` bash
  packetbeat setup
```
  ![](Images/Packetbeat%20setup.png)

Then restarted all 3 services by using systemctl restart to enable all 3 services.

  ![](Images/systemctl%20restart.png)

### Capstone vulnerable VM exploit (Finding the flag)
Discover the IP address of the Linux web server:
``` bash
  nmap -sn 192.168.1.0/24
```
  ![](Images/Linux%20Web%20server%20discovery.png)
``` bash
nmap -A 192.168.1.105 – This shows it is indeed a Linux OS for this machine
And then nmap -sV to discover open ports and running services on the web server
```
  ![](Images/Confirming%20Linux%20Web%20Server.png)
  ![](Images/Services%20and%20ports%20scan%20on%20Web%20Server.png)

Locate the hidden directory on the web server – I manipulated the URL to find the secret_folder directory

  ![](Images/Finding%20hidden%20directory%20on%20web%20server.png)

Brute force the password for the hidden directory using the hydra command:
``` bash
First, I went into /usr/share/wordlists and I ran gunzip rockyou.txt.gz to unzip the rockyou.txt file. Then I ran hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 22 -f -vV 192.168.1.105 http-get 192.168.1.105/company_folders/secret_folder/. I know ashton was the user because after looking around in the company directories I found a file stating that she is in charge of credit card info located in that directory.
```
  ![](Images/Hydra%20brute%20force.png)
  ![](Images/Logged%20into%20hidden%20directory.png)

Break the hashed password with the Crack Station website or John the Ripper (I created text file called hash.txt with the hashed passwd found on webserver)
``` bash
Ran john --format=raw-md5 hash.txt
Alternatively you could use crackstation.net to achieve the same
```
  ![](Images/Cracking%20hashed%20password%20with%20john%20the%20ripper.png)
  ![](Images/Cracking%20hashed%20password%20with%20Crackstation.png)

Connect to the server via WebDav
``` bash
Found instructions to connect to the server via WebDav in the secret directory.
Then uploaded a php reverse shell payload that I created using MSVenom
Finally, executed payload with help of metasploit to open a meterpreter session
```

  ![](Images/Instructions%20to%20connect%20to%20webdav%20server.png)
  ![](Images/Accessing%20WebDav%20through%20File%20Manager.png)
  ![](Images/MSVenom%20PHP%20reverse%20shell%20payload.png)
  ![](Images/Uploading%20PHP%20exploit%20onto%20WebDav.png)
  ![](Images/metasploit%20exploit%20set%20up.png)
  ![](Images/metasploit%20exploit%20execution.png)

Find and capture the flag

  ![](Images/Finding%20the%20flag%201.png)
  ![](Images/cat%20flag.txt.png)






