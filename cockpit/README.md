# Cockpit Machine from brCTF

## Information Gathering/Reconnaissance <br>
As usual we fire up our favourite network mapper (nmap) and perform some basic recon. We can see that we have only port 80 (http) open.
`nmap -sC -sV -oN cokcpit.nmap 10.0.160.145`
```
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

In the second stage of out recon we checkout the application running on port `80` from our browser. <br>
We are greeted with a homepage.... interesting!
![Cockpit Homepage](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/homepage.png "a title")

Next we try to find out what other pages are available on our target. We employ the servies of our directory bruteforcing tool (feroxbuster) or an other webfuzzer. <br>
`feroxbuster -u http://10.0.160.145 -w /raft-medium-wordlists.txt`
```
302      GET        0l        0w        0c http://10.0.160.145/ => http://10.0.160.145/auth/login?to=%2F
302      GET        0l        0w        0c http://10.0.160.145/system => http://10.0.160.145/auth/login?to=%2Fsystem
400      GET        7l       11w      157c http://10.0.160.145/!%22%C2%A3$%^
302      GET        0l        0w        0c http://10.0.160.145/?????? => http://10.0.160.145/auth/login?to=%2F%3F%253F%253F%253F%253F%253F%3D
400      GET        7l       11w      157c http://10.0.160.145/100%sexy
400      GET        7l       11w      157c http://10.0.160.145/100%angel
400      GET        7l       11w      157c http://10.0.160.145/100%bitch
400      GET        7l       11w      157c http://10.0.160.145/100%me
400      GET        7l       11w      157c http://10.0.160.145/100%cute
400      GET        7l       11w      157c http://10.0.160.145/100%love
400      GET        7l       11w      157c http://10.0.160.145/!%22%C2%A3$%^&*()
302      GET        0l        0w        0c http://10.0.160.145/content => http://10.0.160.145/auth/login?to=%2Fcontent
400      GET        7l       11w      157c http://10.0.160.145/100%cool
301      GET        7l       11w      169c http://10.0.160.145/install => http://10.0.160.145/install/
404      GET        0l        0w        0c http://10.0.160.145/austin101
```

Something interesting the results above. We find a `/install` endpoint. <br>
After visiting this page in the browser we realized that the system administrator setup cockpit application without fully installing. Nice! we can abuse this. <br> After visiting this endpoint cockpit will be installed and will setup an account with default logins.
![Cockpit Install Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/install_the_software.png "a title")

Using the default logins obtained on installation we can login to the cockpit application.
![Cockpit Login Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/login_with_default_creds.png "a title")

Successful Login attempt and we a greeted with the application dashboard. From there we can do a little recon to find out the version of cockpit running. From the image below, `cockpit v2.4.0` is found to be running.
![Cockpit Dashboard Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/cockpit_version_info.png "a title")

As a hacker google should be one of your cherised tools. Using the google search engine we can search for publicly available exploit for `cockpit v2.4.0`. <br> We find a publicly disclosed exploitation process [here](https://huntr.dev/bounties/f73eef49-004f-4b3b-9717-90525e65ba61/).
![Public Expliot](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/google_vuln.png "a title")

From the steps provided on the [website](https://huntr.dev/bounties/f73eef49-004f-4b3b-9717-90525e65ba61/).
1. We must first craft out malicous php script.
![exploit code](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/exploit_code.png "a title")

2. Head over to the assest page on cockpit and upload our malicious php script.
![uploading php script](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/uploading_a_malicious_php_script_huntrdev.png "a title")
![uploading php script](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/uploading_php_script.png "a title")

