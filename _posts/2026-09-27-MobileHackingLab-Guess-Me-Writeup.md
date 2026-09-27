---
title: "Mobile-Hacking-Lab : 'Guess Me' mobile challenge  "
date: 2026-09-27 13:22:00 +0000
categories: [Writeups, mobile]
tags: [jadex, android-pentesting,mobile-application, CTF , js interface, deep link vulnerability, RCE]
description: "An easy mobile challenge from Mobile Hacking Lab introducing how to exploit a deep link vulnerability and get an RCE"
image:
  path: /assets/images/writeups/mobile/Guess-Me/guess-me-logo.png
  alt: ""
---


# Challenge Overview :
the Geuss-Me application is designed to be vulnerable to deep link and an RCE  vulnerabilities , the app load pages within a WebView which leads to RCE 
, in order to solve this challenge you need to have basic skills in reading java code and basic exploitation skills for deep link and getting RCE (the two vulnerabilities are separate, you don't need to chain them in order to solve the challenge)

# Hacking the Application 

my methodology begins with exploring the app as a normal user so i can get better understanding of how the app works and functions , let's open the app and see what is waiting for us 

## Exploring the application

as we can see when we open the app we interfere with the main activity 

![ main activity  ](/assets/images/writeups/mobile/Guess-Me/main-activity.png)

from the image we can get an overview about what the app does , it is a guessing game , we enter a number and it tel's us if we guessed right or wrong 
if we guess the number wrong or right we get message as in the image below

![ wrong guess for the number  ](/assets/images/writeups/mobile/Guess-Me/wrong-guess.png)

![ right guess for the number  ](/assets/images/writeups/mobile/Guess-Me/wrong-guess.png)

from the previous images we can see there is a ? icon in the main activity , upon clicking on this icon we get another activity which is the `WebviewActivity` (as we can see later in the manifest file )

![ webview activity  ](/assets/images/writeups/mobile/Guess-Me/WebviewActivity.png)

what this activity does is providing us with an HTML page that contains the current date and a url to the mobilehackinglab , which is an indicator to it may be vulnerable to deep link vulnerability 
>> to read more about deep link vulnerability , visit this sites: https://nirajkharel.com.np/posts/android-pentesting-deeplinks/ \n https://0xn3va.gitbook.io/cheat-sheets/android-application/intent-vulnerabilities/deep-linking-vulnerabilities


## Static Analysis 

