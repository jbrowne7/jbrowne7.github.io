---
title: "SOC Automation Home Lab"
date: 2024-12-10
media_subpath: /assets/posts/2024-12-10-soc-automation-home-lab
published: false
layout: post
categories:
- home lab
tags:
- soc
- wazuh
- hive
- virustotal
- mimikatz
- shuffle
- soar
- siem
- windows
---

## Demo

{% include embed/youtube.html id="GFqHj3MzNSI" %}

## Brief Summary

For this project we'll be using Wazuh for our SIEM and EDR tool, Shuffle for our SOAR tool, and TheHive for case management. Here's the setup diagram which may be slightly confusing at first

![SOC Automation home lab project](SOC_automation_project_diagram.png)

This diagram lays out the flow of workflow which will take place, in more simpler terms:
 - Our SIEM (Wazuh manager) receives alerts from log sources (Wazuh agents)
 - The SIEM then triggers alerts which our sent to our SOAR (Shuffle)
 - Our SOAR then utilises OSINT to enrich any IOCs and also alerts the analyst and opens a case in our case management system (The Hive)
 - From here the analyst makes a decision on what to do and sends it to the SOAR (Creating new workflow for this, not included in demo yet)
 - The SOAR can then execute these actions, e.g telling Wazuh to quarantine a file (Creating new workflow for this, not included in demo yet)

--Need to finish report for this project--
 
