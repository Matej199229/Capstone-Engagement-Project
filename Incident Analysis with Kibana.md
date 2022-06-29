# Blue Team: Incident Analysis with Kibana

## Table of Contents
- Adding Kibana Log Data
- Adding Apache and System Metrics
- Dashboard creation with reports
- Identifying offensive traffic
- Finding the request for the hidden directory
- Identifying the brute force attack
- Finding the WebDav connection
- Identifying the reverse shell and meterpreter traffic

### Adding Kibana Log Data

To start viewing logs in Kibana, we will need to import our filebeat, metricbeat and packetbeat data.
``` bash
Adding Apache logs and System logs:
```
  ![](Images/Apache%20logs%201.png)
  ![](Images/Apache%20logs%202.png)
  ![](Images/Apache%20logs%203.png)

  ![](Images/System%20logs%201.png)
  ![](Images/System%20logs%202.png)
  ![](Images/System%20logs%203.png)


### Adding Apache and System Metrics

``` bash
Adding Apache and System metrics:
```
  ![](Images/Apache%20Metrics%201.png)
  ![](Images/Apache%20Metrics%202.png)
  ![](Images/Apache%20Metrics%203.png)

  ![](Images/System%20Metrics%201.png)
  ![](Images/System%20Metrics%202.png)
  ![](Images/System%20Metrics%203.png)

### Dashboard creation with reports

``` bash
Creating a Kibana dashboard using the pre-built visualizations. 
```

  ![](Images/Dashboard%20creation.png)
  ![](Images/Reports%20created%20in%20Dashboard.png)


### Identifying offensive traffic

``` bash
When did the interaction occur? 
```

  ![](Images/When%20did%20offensive%20traffic%20occur.png)

May 14th, 2022 between 15:50 and 15:55 was when the majority of the interaction occurred.

``` bash
What responses did the victim send back?  
```
  ![](Images/What%20responses%20victim%20sent%20back.png)
  ![](Images/What%20responses%20victim%20sent%20back%202.png)

207 multi status responses were sent back to the attacker.

``` bash
What data is concerning from the Blue Team perspective?  
```
  ![](Images/Concerning%20data%20for%20Blue%20team.png)

There is an abnormal amount of traffic coming from the same source_IP 192.168.1.90 to the web server in a very short time period over port 80 indicating suspicious activity.


### Finding the request for the hidden directory

``` bash
How many requests were made to this directory? At what time and from which IP address(es)?   
```
  ![](Images/Requests%20to%20hidden%20directory.png)
  ![](Images/Requests%20to%20hidden%20directory%202.png)

There were 16444 http get requests to this directory between 15:51:55 and 15:53 from the same IP address 192.168.1.90.

``` bash
Which files were requested? What information did they contain?    
```
  ![](Images/Requested%20files%20in%20hidden%20directory.png)

The secret folder directory was requested which contained the instructions for how to obtain access to the corp webdav server in the connect_to_corp_server file.

``` bash
What kind of alarm would you set to detect this behavior in the future?     
```

I would create an alarm based on a baseline for normal http get requests to this directory 
and threshold each 30 mins for example. In this case it would be if there is more than 0 get requests to this directory, there should be an alarm sent out to the SOC management as NO ONE should be accessing this directory externally. 

``` bash
Identify at least one way to harden the vulnerable machine that would mitigate this attack.
```

- Improve the password for accessing this directory
- Blacklist source IP 192.168.1.90 with a firewall rule
- Remove the directory. Why is a secret folder directory needed on a web server? This information should be stored on a standalone and much more secure web server if it is truly of secret and of utmost importance.
- Files and folders should be encrypted.

### Identifying the brute force attack

``` bash
Can you identify packets specifically from Hydra? 
```

  ![](Images/Hydra%20brute%20force.png)

``` bash
How many requests were made in the brute-force attack? 
```

16428 HTTP GET requests in total with the GET /company_folders/secret_folder/ query and from user agent Mozilla 4.0/(Hydra)

``` bash
How many requests had the attacker made before discovering the correct password in this one?  
```
  
  ![](Images/Attacker%20requests%20before%20guessing%20correct%20password.png)

16426 requests with status marked as “error”

``` bash
What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?   
```

The alarm I would set would be based on ANY requests (more than 0) coming in to the web server from user agent Hydra as this is a known tool that hackers use to brute force into web applications.

``` bash
Identify at least one way to harden the vulnerable machine that would mitigate this attack.   
```

- Complex and long passwords to make it harder for brute force attacks to succeed
- Blacklist IPs associated with brute force attacks

### Finding the WebDav connection

``` bash
How many requests were made to this directory?    
```
  ![](Images/Requests%20to%20WebDav%20directory.png)

84 requests to the /WebDav directory in total

``` bash
Which file(s) were requested?     
```

Exploit.php

``` bash
What kind of alarm would you set to detect such access in the future?     
```

Whitelist specific machines that are allowed access. Alarm should be set for any machine trying to access webdav that is not on the whitelist

``` bash
Identify at least one way to harden the vulnerable machine that would mitigate this attack.     
```

Whitelist machines that are allowed access to webdav

### Identifying the reverse shell and meterpreter traffic

``` bash
Can you identify traffic from the meterpreter session?     
```

  ![](Images/Reverse%20shell%20and%20meterpreter%20traffic.png)

I searched by source.port = 4444 as that is the port that the meterpreter session is being directed from the attacker machine. We have 116 counts on that port.

``` bash
What kinds of alarms would you set to detect this behavior in the future?     
```

I would set an alarm for incoming traffic from source port 4444. If we are not expecting ANY normal traffic from this port otherwise, then the alarm should be triggered if there is even 1 count of traffic from this port onto the server as it could be suspicious since port 4444 is the default listening port used for meterpreter sessions.

``` bash
Identify at least one way to harden the vulnerable machine that would mitigate this attack.    
```

- Using tools like Antipwny or Antimeter. These tools help find and kill the meterpreter session that the attacker has gained.
- Run applications/processes on server with least privileges
- Limit network access to only trusted hosts
