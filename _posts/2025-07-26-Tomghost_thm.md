---
title: "TryHackMe: Tomghost"
date: 2025-07-16
categories: [writeup]
tags: [THM,Easy]
medium_url: "https://medium.com/@abouzennar/%E2%84%8D-e2f1453ad15f"
---

## Tomghost THM Write-up

📖 **Full walkthrough on Medium:** [Tomghost Write-up]({{ page.medium_url }})

### Summary:
**Difficulty:** Easy

Exploited Tomcat 9.0.30 vulnerability leading to:
- SSH access via discovered credentials
- PGP key cracking with John the Ripper
- Privilege escalation via zip binary (GTFOBins)
- Root access and flag capture

**Key steps:**
1. Nmap scan revealed vulnerable Tomcat
2. Metasploit exploit for file inclusion
3. PGP credential cracking
4. GTFOBins zip exploit for root

**Room:** [Tomghost](https://tryhackme.com/room/tomghost)

