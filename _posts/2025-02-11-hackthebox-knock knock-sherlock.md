---
title: "Hackthebox Knock Knock Lab"
date: 2025-02-11
media_subpath: /assets/posts/2025-02-11-hackthebox-knock knock-lab
categories:
- ctf
- blue team
tags:
- linux
- hackthebox
- wireshark
- blue team
- pcap
published: false
layout: post
---


Scenario:  

"A critical Forela Dev server was targeted by a threat group. The Dev server was accidentally left open to the internet which it was not supposed to be. The senior dev Abdullah told the IT team that the server was fully hardened and it's still difficult to comprehend how the attack took place and how the attacker got access in the first place. Forela recently started its business expansion in Pakistan and Abdullah was the one IN charge of all infrastructure deployment and management. The Security Team need to contain and remediate the threat as soon as possible as any more damage can be devastating for the company, especially at the crucial stage of expanding in other region. Thankfully a packet capture tool was running in the subnet which was set up a few months ago. A packet capture is provided to you around the time of the incident (1-2) days margin because we don't know exactly when the attacker gained access. As our forensics analyst, you have been provided the packet capture to assess how the attacker gained access. Warning : This Sherlock will require an element of OSINT to complete fully."  

For this sherlock we are given a pcap file, lets open it in wireshark and start the analysis.  

### Question 1

**Which ports did the attacker find open during their enumeration phase?**  

To answer this question a good place to start would be assuming that the attacker performed a port scan of the server, its likely that the server received a lot of packets to various ports which wouldn't usually be accessed.