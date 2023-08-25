---
title: "TryHackMe Bookstore Writeup"
datePublished: Wed Aug 02 2023 14:40:25 GMT+0000 (Coordinated Universal Time)
cuid: clktu4tut00010amlgxglgrfa
slug: tryhackme-bookstore-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692952874743/eab0eb71-a767-4394-9795-13873ae0fc3c.jpeg
tags: tryhackme, write-up, enumeration, fuzzing

---

### Start

After starting the victim machine I started my Parrot VM on VirtualBox connected via OpenVPN to the tryhackme network and checked if I was really connected. With a Ping, I ensured that it was visible and the connection was established.

### Enumeration

First I started with `nmap [victim machine IP]` It shows a possible AIP on port 5000, A second nmap run `nmap -sC -sV [victim machine IP]` gives a lot more detail. I was curious when I saw Werkzeug on port 5000. It seems some sort of a debugger was involved with a Patreon Hack in 2015. But first I tried every button and function on http://\[victim machine IP\]/ with no results, everything is basically a dummy. I searched with Gobbuster for interesting sub-directories but had no luck. A quick

```plaintext
gobuster dir -u http://[victim machine IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

shows nothing interesting. I checked http://\[victim machine IP\]:5000, and it reveals the used API called Foxy API. I Google searched the documentation of the API, but there wasn't anything helpful here. There was some kind of exploit with this API but I haven't tried that. Then I did another gobuster scan, this time on \[victim machine IP\]:5000.

```plaintext
gobuster dir -u http://[victim machine IP]:5000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

The scan gave me with /api and /console two interesting sub-directories which I check out immediately. http://\[victim machine IP\]:5000/api/ gave me some Documentation, a nice find to play around with. http://\[victim machine IP\]:5000/console/ on the other hand gave me, what seems like a Python console in the browser. Unfortunately, the console is locked with a PIN code.

### Gaining initial foothold

I tried to play around with the findings on the API Documentation page. But again had no luck. I tried wfuzz on the API's but it showed nothing new. While playing around I noticed /api/v2/ in the URl's and I wondered if v2 = version 2 and if so, some rest of version 1 still exists somewhere. So I kept fuzzing around until I had success with

```plaintext
wfuzz -u http://[vicitim machine IP]:5000/api/v1/resources/books?FUZZ=1 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404
```

which gave me a new parameter that I could try out. Maybe LFI (Local File Inclusion) is working, so I tried to get the bash history http://\[victim machine IP\]:5000/api/v1/resources/books?show=.bash\_history which actually worked and gave me the needed pin for the console.

After I inserted the pin I had access to a Python console to play with. A reversed shell would be much appreciated now. I set up a Netcat listener on port 1234 on my attack machine and started Googling for a Python reversed shell. I found a a nice manual here: [Python\_Shell](https://www.linuxfordevices.com/tutorials/shell-script/reverse-shell-in-python) And was granted a shell as sid. With `ls` I found the first flag in user.txt

### Becoming Root

The other files where mostly uninteresting except for the try-harder. Trying `./try-harder` gave me a prompt to insert the magic number which I didn't have, so I guess I have to try harder... I kept searching for clues what the magic number might be, but there were none. Then I opened a simple Python server on the victim machine and wget it to my attack machine. I knew that with Ghidra you can reverse engineer Software. I didn't have any experience with Ghidra or reverse engineering. As many times, YouTube for the rescue. This [Ghidra\_Tutorial](https://www.youtube.com/watch?v=fTGTnrgjuGA&t=488s) gave me enough information to reverse engineer the try-harder script. The magic number is the result of three XOR'd HEX numbers found in the code. Running the try-harder script again on the victim machine and inserting the magic number granted me root. in /root/root.txt I found the root and final flag.