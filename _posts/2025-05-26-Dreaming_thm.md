---
title: "TryHackMe: Dreaming"
date: 2025-05-26
categories: [writeup]
tags: [THM,Easy]
medium_url: "https://medium.com/@abouzennar/tryhackme-walkthrough-dreaming-d74da17f7142"
---

## Dreaming THM Write-up

📖 **Full walkthrough on Medium:** [Dreaming Write-up]({{ page.medium_url }})

### Summary:
**Difficulty:** Easy

Multi-step CTF focusing on:
- Pluck CMS 4.7.13 exploitation (RCE via upload)
- MySQL injection for privilege escalation
- Python library hijacking (shutil.py) for root access
- Lateral movement between users (lucien → death → morpheus)

**Key techniques:**
1. CMS version exploitation
2. Database manipulation via SQL
3. Python library hijacking
4. Lateral movement through 3 users

**Tools used:** nmap, gobuster, Python RCE exploit, MySQL


*Multi-user CTF with CMS exploitation and Python privesc techniques.*
