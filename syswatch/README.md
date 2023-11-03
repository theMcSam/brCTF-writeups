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

