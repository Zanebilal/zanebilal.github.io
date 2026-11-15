---
title: "The biggest EDR Nightmare : Basic EDR Killer "
date: 2026-11-13 15:07:00 +0000
categories: [Red Team, EDR , Red Team , Evasion]
tags: [process, windows-internals, malware-analysis,malware-dev, EDR, kernel , evasion,]
description: "one way shot used by hacker to avoid edr detection is to kill them right a way without thinking on new evasion techniques, this way the deployed malware will move freely in the system without detection because there is now guard , no detection , no monitor that can stop it "
---

>Hi, I'm **Zane Bilal** — a computer science student and explorer of the offensive side of cybersecurity. In the world of malware and low-level programming, understanding *how things are built* is non-negotiable. to see my work explore my github and my other posts below 

>> `Disclamer:` what you learn from this blog and how you use the knowledge gained from it is up to you , i am not responsible from any misuse of the code in this blog , just use it for educational purposes

# Why You need to care about EDR Killers : 

you are in the field of cybersecurity as a student , a researcher, an employee ..etc , you need to know about edr killer and how much they are so powerful.
so basically an EDR Killer is a rootkit that lives in the kernel that has the least privileges over the system which makes them able to do anything in the machine and main purpose is to terminate the EDR process so the system will not be monitored by any edr OR Anti-virus , and in this point you are cooked and you have now idea about the termination of your edr.

#  What is an "EDR Killer"
as i mentioned before , a program that runs in the kernel (with kernel privileges) are called rootkit and an EDR killer is one of them and as its name suggest , it purpose to kill EDR and anti-virus processes , but for modern malware this technique is not used alone rather it is used along with other techniques to evade edr before killing it like etw and amsi bypass .. etc
and the question to ask is why attacker targets edr processes and use this technique specifically and not other evasion techniques and methods as i mentioned before? 
to answer this question we need to understand how the windows detect suspicious softwares , and a simple answer is by collecting the system telemetry and compare it with already flagged ones, and the primary source of this telemetry is EDR process , additional to that EDR processes runs under system privileges so targeting them is a primary goal 

# How EDR Killers are Implemented 

* before we start the technical part of this blog , there is some prerequisite needed from you to know to continue this post , ad some of them is to know how edr processes collect telemetry and which win api's used for that and also how to code a basic driver and how it communicates with a userland program (sending and receiving data from and to the driver)

* 



