---
title: "TryHackMe: Windows Server Forensics"
date: 2025-04-15
categories: [writeup]
tags: [THM, Forensics, Windows, SOC]
---

## Windows Server Forensics  Complete Walkthrough

This room simulates a real world forensic investigation of a compromised Windows Server. Using only native Windows tools (PowerShell, Registry Editor, Event Viewer, and the command line), we'll uncover persistence mechanisms, identify the attacker's actions, and piece together the timeline of the breach.

 Let's dive in.

 
![gif 1](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcGViOGJ1NjBjNjIxc3lzbWxqMmh3Y3hjb3I3YnJyb2hpMzRuMW12dSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YQitE4YNQNahy/giphy.gif)

---

### 1. Identifying the System

**Question:** *Whats the version and year of the windows machine?*

We start by opening PowerShell. The `Get-ComputerInfo` cmdlet reveals everything about the system, but that's too much information. Let's filter for OS-related properties only:
**"Get-ComputerInfo -Property "Os"**
![image 1](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20192149.png?raw=true)

Answer: Windows Server 2016
### 2. User Login Analysis
**Question:** Which user logged in last?

First, let's see all local users on the system:
**Get-LocalUser**
![image 2](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20192514.png?raw=true)

Now we need to check each user's last login. The net user command with `findstr` filters for the "Last" logon field:

**net user Administrator | findstr "Last"
net user John | findstr "Last"
net user Jenny | findstr "Last"**
![image 3](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20192528.png?raw=true) 

The Administrator account shows the most recent login.

Answer: Administrator
### 3. John's Last Logon
**Question:** When did John log onto the system last?
Format: MM/DD/YYYY H:MM:SS AM/PM
From the output of net user John | findstr "Last", we get the timestamp. Just remember to add leading zeros for the month and day.
**Answer: 03/02/2019 5:48:32 PM**

### 4. Administrative Privileges
**Question:** What two accounts had administrative privileges (other than the Administrator user)?
Format: List them in alphabetical order.

We can check group membership using PowerShell:

**Get-LocalGroupMember -Group "Administrators"**
The output shows all members of the local Administrators group. Excluding the built-in Administrator account, we find two others. Sort them alphabetically.
![image 4](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20193814.png?raw=true) 

Answer: Guest, Jenny

### 5. Jenny's Last Logon
**Question:** When did Jenny last logon?

Back to the net user command:
**net user Jenny | findstr "Last"**
![image 5](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20192528.png?raw=true)

Answer: Never
#### Part 2: Persistence & Network Artifacts
### 6. Startup Connection IP
**Question:** What IP does the system connect to when it first starts?

This requires a bit of digging. First, I checked the hosts file for anything suspicious:

Path: `C:\Windows\System32\drivers\etc\hosts`
Opening it in Notepad revealed some odd entries Google domains pointing to strange IPs. That's DNS poisoning. But for the startup IP, we need to check the Registry.

Open Regedit (Windows key + type "regedit") and navigate to:
`HKEY_LOCAL_MACHINE > SOFTWARE > Microsoft > Windows > CurrentVersion > Run`
Two values are present. The UpdateSvc entry looks suspicious. Double-clicking it reveals an IP address.

Answer: 10.34.2.3
### 7. Malicious Scheduled Task
**Question:** Whats the name of the scheduled task that is malicious?

Let's list all scheduled tasks:
`Get-ScheduledTask`
That's a long list. Let's filter for tasks in the root folder only:

`Get-ScheduledTask | where {$_.TaskPath -eq "\"}`
This narrows it down to about six tasks. One name stands out as unusual.
![image 6](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20194306.png?raw=true)

Answer: Clean file system
### 8. Task Execution Details
**Question:** What file was the task trying to run daily?
Let's store the task in a variable and inspect its actions:
`$task = Get-ScheduledTask | Where TaskName -EQ "Clean file system"`
`$task.Actions`
**The output shows the executable being run.**
![image 7](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20194803.png?raw=true)

Answer: **nc.ps1**
### 9. Listening Port
**Question:** What port did this file listen locally for?
Looking at the `$task.Actions` output from above, the Arguments column contains the port number.
**Answer: 1348**
##### Part 3: Timeline & Event Analysis
### 10. Date of Compromise
**Question: At what date did the compromise take place?**
Open File Explorer and navigate to `C:\`. Look at the folder creation dates. One date appears multiple times across suspicious folders that's our compromise date. Add leading zeros.
![image 8](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20195015.png?raw=true)
    
Answer: 03/02/2019
### 11. Privilege Assignment Time
**Question: During the compromise, at what time did Windows first assign special privileges to a new logon?**

This requires Event Viewer. Open Event Viewer (Start → type "Event Viewer").
Click Create Custom View on the right. Set up a custom range:
Logged: Custom Range

From: 03/02/2019 4:00:00 PM
To: 03/02/2019 4:30:00 PM

Event Logs: `Windows Logs → Security`

Click OK and save the filter. Now browse through the events. Look for Task Category: Security Group Management this indicates privilege changes for a new logon. The timestamp on that event is our answer.
Answer: 03/02/2019 4:04:49 PM

##### Part 4: Attacker Tools & Infrastructure
### 12. Password Dumping Tool
**Question: What tool was used to get Windows passwords?**

A suspicious process kept popping up: `C:\TMP\mim.exe`. Navigate to `C:\TMP` in File Explorer. Inside, you'll find a file named mim-out. Opening it reveals the tool name.
![image 9](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20200709.png?raw=true)
Answer: Mimikatz
### 13. C2 Server IP
**Question: What was the attackers external control and command servers IP?**

Remember the hosts file we looked at earlier? Open it again:
`C:\Windows\System32\drivers\etc\hosts`
![image 10](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20200936.png?raw=true)
Answer: 76.32.97.132 
##### Part 5: Web Shell & Firewall Backdoor
### 15. Web Shell Extension
**Question: What was the extension name of the shell uploaded via the servers website?**
Navigate to the IIS web root:
`C:\inetpub\wwwroot`
Inside, you'll see three files. Two have an extension commonly used for Java based web shells.
![image 11](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20201329.png?raw=true)

Answer: .jsp
### 16. Last Port Opened by Attacker
**Question: What was the last port the attacker opened?**

Open Windows Defender Firewall with Advanced Security (Start → type "firewall").
Click Inbound Rules on the left. Then click Filter by Group on the right and select Rules without a Group.
Two rules remain. One looks suspicious: Allow outside connections for development. Double-click it, go to the Protocols and Ports tab, and look at the Local Port.
![image 12](https://github.com/AnouarTheg1/media/blob/main/Capture%20d'%C3%A9cran%202026-04-15%20201547.png?raw=true)
 
 **Answer: 1337**

![gif 2](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcHpua2Frb2Uwd2NkaGY4bG9ydzR2YTVlYncycWYwcTN5djZpZnRzdyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/j4rPM934CLIvC/giphy.gif)


Room:              `https://tryhackme.com/room/investigatingwindows`
