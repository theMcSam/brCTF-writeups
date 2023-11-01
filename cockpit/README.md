# Cockpit Machine from brCTF

## Information Gathering/Reconnaissance <br>
As usual we fire up our favourite network mapper (nmap) and perform some basic recon. We can see that we have only port 80 (http) open.
```
nmap -sC -sV -oN cokcpit.nmap 10.0.160.145
Nmap scan report for 10.0.160.145
Host is up (0.24s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
| http-title: Cockpit
|_Requested resource was /auth/login?to=%2F

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

In the second stage of out recon we checkout the application running on port 80 from our browser. <br>
![Cockpit Homepage](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/homepage.png "a title")
