---
title: "Hackthebox Jinkies Sherlock"
date: 2025-02-06
media_subpath: /assets/posts/2025-02-06-hackthebox-jinkies-sherlock
published: false
layout: post
---

Given scenario:  

"You’re a third-party IR consultant and your manager has just forwarded you a case from a small-sized startup named cloud-guru-management ltd. They’re currently building out a product with their team of developers, but the CEO has received word of mouth communications that their Intellectual Property has been stolen and is in use elsewhere. The user in question says she may have accidentally shared her Documents folder and they have stated they think the attack happened on the 6th of October. The user also states she was away from her computer on this day. There is not a great deal more information from the company besides this. An investigation was initiated into the root cause of this potential theft from Cloud-guru; however, the team has failed to discover the cause of the leak. They have gathered some preliminary evidence for you to go via a KAPE triage. It’s up to you to discover the story of how this all came to be. Warning: This sherlock requires an element of OSINT and players will need to interact with 3rd party services on internet."  

Key points to extarct:  
- IP has been stolen from a company (cloud-guru-management ltd) due to a user sharing her documents folder
- They think attack happened on 6th of October
- We have some digital evidence (KAPE triage) to investigate what happened

### Question 1

**Which folders were shared on the host? (Please give your answer comma separated, like this: c:\program files\share1, D:\folder\share2)**  

- Attack occured on 6th of october so we'll start with looking at events from this day
- We want to find some logs which will show us evidence of some form of file transfer, lets start with network information from 