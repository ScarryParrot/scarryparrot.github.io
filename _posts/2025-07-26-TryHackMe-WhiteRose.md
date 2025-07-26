---
title: TryHackMe - WhiteRose
date: 2025-07-26
categories: [CTF, THM]
tags: [Linux,Ejs,sudoedit]
---

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/cover.png)

This is an easy machine from TryHackMe. We started with nmap and found 22 and 80 as open ports. Doing a subdomain enumeration we found a misconfiguration exploiting which we gained password for other user. Logging in we exploited an ejs injection vulnerability leading to RCE. For the root part exploiting a misconfigured sudoeditor file we get the root flag.

Starting off with rustscan we find port 80 and 22 open. These had nothing on yet just a blank site. 

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/scan.png)

So upon futher enumeration, we found a domain **cyprusbank.thm** from the header. So do we stop here? No. We have to check for other subdomains.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/domain.png)

Firstly, we added them to out host file, **/etc/hosts** adn used ffuf for subdoamin enumeration. We have to filter out words of length 1 to keep our terminal clean.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/ffuf.png)

add them too in the host file and on **admin.cyprusbank.thm** we found a login page. We can login as
> *And oh! I almost forgot! - You will need these: Olivia Cortez:olivi8*

>These cerds were given at the beginning.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/web.png)

We got in but seems like we cant do much. We need to elevate our privilage. Checking all the links we see a message functionality. Teaking the url we can view older messages and get a password.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/message.png)

Upon loggin in as Gyale, there was a settings functionality which adds user.So we intercepted the request using Burp. When removing one parameter it gives error indicating ejs injection. A quick lookup on google provided us with the exploit.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/ejs.png)

>[Github Link](https://github.com/mde/ejs/issues/735?source=post_page-----fbe6a2bf70af---------------------------------------)

I tried many payloads , none worked, they all gave error. But using busybox we can get a rce.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/rce.png)

We can now get the user flag. To vertically elevate out privilage we ran **sudo -l**. And we can sudoedit a file. Upon looking a sudoedit exploit I found it was a misconfiguration. 
>[Blog](https://www.vicarius.io/vsociety/posts/cve-2023-22809-sudoedit-bypass-analysis)

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/sudo.png)

This blog talks about it ad following the steps we can edit any file we like. I went for /etc/sudoers and set it to all.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/sudo2.png)

And we got root flag and a shell as root.

![whiterose thm](/Images/2025-07-26-TryHackMe-WhiteRose/root.png)
