---
title: "CyberDefenders Kerberoasted Lab"
date: 2024-10-07
published: false
layout: post
---

Information from reading the scenario:
- We are an analyst performing an investigation to see if our organization is vulnerable to kerberoasting attacks
- We begin with an in depth analysis of the DC logs

First we need to understand what is a kerberoasting attack, from reading this [article]("https://www.crowdstrike.com/cybersecurity-101/kerberoasting/") we see that kerberoasting is a post exploitation attack which involves capturing the password hash of an AD account that has a SPN, an SPN is an attribute which ties a service to a user account within the AD. In the attack the authenticated user requests a ticket for an SPN and the retrieved kerberos ticket is encrypted with the hash of the serivce account password affiliated with the SPN, through this the attacker obtains the password hash for the account.


### Question 1

We need to find the encryption type used within the network. Looking at all the fields in splunk and searching the word "encryption" we find a field `winlog.event_data.TicketEncryptionType`, and we also see that there is only one value for this field, `0x17`. When we filter for this and look at all the possible fields again we see that there are two possible winlog event id's, 4768 and 4769, a quick google search for winlog 4768 tells us that this log is generated every time Key Distribution Center issues a Kerberos Ticket Granting Ticket (TGT). In the contents section of the article there is a "Kerberos encryption types" section, and this then gives us the table for the corresponding types, revealing the answer for `0x17` to be the encryption protocol RC4-HMAC

### Question 2

check log for id corresponding to TGS request, we get from Q1, event id 4768, filter for this,
