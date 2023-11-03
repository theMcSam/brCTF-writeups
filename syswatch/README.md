# Write Up on syswatch machine

## Information Gathering And Enumeration
Let's us first start of by firing up our network mapper (nmap) to discover open ports and running services on our target.
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

Let's take a look at it in our browser.
![zabbix homepage](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/zabbix-port-5000-browser.png)

We are greeted with a login page and as usual we test default credentials to see if they work. After some googling we come across the default credentials for `zabbix`.
![zabbix default creds](https://raw.githubusercontent.com/theMcSam/brCTF-writeups/main/syswatch/images/default-logins-zabbix-google.png)