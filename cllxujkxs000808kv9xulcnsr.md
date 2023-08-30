---
title: "TryHackMe Wonderland Writeup"
datePublished: Wed Aug 30 2023 14:42:40 GMT+0000 (Coordinated Universal Time)
cuid: cllxujkxs000808kv9xulcnsr
slug: tryhackme-wonderland-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1693406396546/6a0aa5f0-1131-47ea-9328-2fa35b494297.jpeg
tags: tryhackme, privilegeesclation, ctf-writeup, path-hijacking

---

### Start the machine

I started by booting up my parrot OS machine on VirtualBox, connected via OpenVPN to the TryHackMe network. After I checked if I was connected I started the Wonderland machine. After the machine IP was available I checked with a Ping if the machine was reachable in the network

```bash
Ping 10.10.145.137
```

### Recon

As usual, I try to gather initial information with Nmap.

```bash
nmap -sC -sV 10.10.145.137
```

After the scan was finished I saw that there were two open ports. The first port 22 for an SSH connection and the second port 80 for HTTP.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692977064292/888a7f78-17e7-4cc9-bea0-e61d9aaeffc7.png align="center")

Without any credentials, the open SSH port is, for now, uninteresting to us. So I decided to follow the white rabbit and gave the http site a visit. I was mildly underwhelmed, to say the least. There was nothing on that page but a picture of the white rabbit and some static text. I checked the source code, but nothing. I tried the usual robots.txt, but nothing was there either. The last straw to catch up with the white rabbit was to find some useful directories with Gobuster.

```bash
gobuster dir -u 10.10.145.137 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

After some time I got this from my gobuster scan.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692977887637/a78282d5-9d61-4dad-8fe9-f12c3d64e7d8.png align="center")

I checked the img/ directory, and inside I found 3 pictures 2 .jpg's and 1 .png. The poem/ directory gave me a poem about the Jabberwocky. The r/ directory just told me "Keep Going" in plain text.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693320490620/86d9d039-86f7-4453-ae35-f055a9051260.png align="center")

I inspected the source of all directories for something interesting. But there was nothing to find. Some time ago I did a CTF on [https://picoctf.org/](https://picoctf.org/). This CTF was about hiding files/text in other files like pictures. I used Steghide to extract/hide information from the sources picoCTF gave me. Since I had no other clue than that I gave it a shot. In a simple example, you're using this command:

```bash
steghide extract -sf [filename]
```

The first picture "alice\_door.jpg" seems to be empty, assuming there's no password to protect the hidden information. The second picture is the same as allice\_door.jpg but as .png. Since Steghide can, as far as I know, only handle .jpg files so I didn't bother to spend time with it. In the third picture "white\_rabbit.jpg", on the other hand, I was able to extract hint.txt.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693319373557/8936fb21-3942-4a05-95a1-0c6b122957fe.png align="center")

Keep Going and follow the rabbit, these are quite confusing hints. I spend quite some time thinking and staring at the screens. Much more than I like to admit. At one point I completely discarded the hint.txt and focused on the Keep Going in /r/. Why not take it literally? I once again run Gobuster. But this time against '10.10.145.137/r'.

```bash
gobuster dir -u 10.10.145.137/r -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Inside /r/ I found another directory /a/.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321305755/7181e061-8f66-4d51-b301-e1ea5c2df4fb.png align="center")

Visiting 'http://10.10.145.137/r/a' told me to Keep going as well.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321441649/67675fa6-5537-4c6d-b8ba-29a3e17cee68.png align="center")

Keep Going, follow the rab... follow the r a b b i t... I immediately tried 'http://10.10.145.137/r/a/b'. And got rewarded with another Keep going. At this point, It was clear what to do.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321657362/31540eb7-53cd-4b9e-8e2b-32ccb82cdf42.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321672771/5838a53d-237c-41f7-981d-e0e864cef589.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321684280/1eb86716-9816-41f4-bc88-429597666026.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693321697860/224176f8-f6a1-4a71-a0eb-76583a136feb.png align="center")

Thats it? Open the door and enter? I inspected the source code of the page and hidden in the code I found a string that could be credentials for a user alice.

### Initial Foothold

Since there's no login or something similar on the machine and the only other port open was for SSH I used the found credentials to log in via SSH.

```bash
ssh@alice 10.10.145.137
```

I entered the password and was granted access as the user Alice.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693322915728/457f0ec8-a6c8-4399-96dc-514d0808087a.png align="center")

I found the root.txt in Alice /home. I was expecting the root flag in /root. I tried to open it but, of course, had no luck. Alice did not have the permission to open root.txt. I used the hint TryHackMe provided to the user flag. It said, "Everything is upside down here". After my ¨follow the rabbit¨ experience I took that hint literally.

```bash
cat /root/user.txt
```

In /root, I found the user flag. I investigated the machine. There were a couple of wonderland-themed users present. Rabbit (of course the rabbit again), Hatter, Alice and TryHackMe. I didn't had permission to view any of them. In Alice's home.

### Privelege Escalation

In addition to the root.txt, there was a Python file. Running the file gave me 10 prompts.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693323694806/4f2cdaf0-3f03-4a6a-a8bc-5a11fd54b40f.png align="center")

I inspected the code of the Python file. There was a poem stored as a string, and a for-loop splitting the poem into lines and randomly choosing 10 lines and printing them on screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693335996617/cc45eef3-a213-414d-8108-4e2881878b83.png align="center")

The code uses the Python random packet which was imported in the first line of the code. I could use the import for a method called path hijacking to escalate my privileges. Python stores installed packets in its directory, but if you choose to write your version of that packet and store it in the same directory as your Python script. Python will choose the version "closer" to its call. But first I had to check if the file is owned by another (more privileged) user. The command 'sudo -l' is showing me that.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693336719654/b21faece-0959-40da-9bdb-51eef271edaa.png align="center")

So Alice can run the file as rabbit with:

```bash
/usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

I wrote a 2 line Python script that spawns a shell.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693336913002/dabd88fd-1274-4cb0-a11f-07150fa35825.png align="center")

To run it as the user rabbit and get a shell as Rabbit I used the following command.

```bash
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

After executing it, I was logged in as Rabbit.

I searched rabbits /home, there was only one file called teaParty.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693337764639/ccd96fbf-7787-4215-863d-cb9966af06e1.png align="center")

Running teaParty as a shell script told me that the Mad Hatter would be here soon. More Interestingly it said 'Segmentation fault (core dumped)'. And 'ls -la' told me the tea party was owned by Root!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693337926001/32c5c464-fc5d-40d4-b250-d890bb43bf91.png align="center")

I chose to investigate the teaParty further. With a simple Python server on the Wonderland machine, I used 'wget' to download that file to my machine. With the 'string' command I investigated the teaParty file. There was one line that caught my interest.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693403893415/e9181aea-edc0-45f6-8746-c91276424a0d.png align="center")

The variable 'date' is not in an absolute PATH. So like with the Python 'random' package, I used this to hijack date. It was as simple as creating a file called 'date' in the same folder as the teaParty file.

```bash
!#/bin/bash
/bin/bash
```

Then I exported '$PATH' to Rabbit.

```bash
export PATH/home/rabbit:$PATH 
```

Now I run the teaParty file again, and we're logged in as Hatter? I was hoping we get root but I made a mistake in the previous step. I was assuming that only because teaParty was owned by Root, I will be granted root access when using it.

### Getting Root

In Hatters home, I found a 'password.txt' file. I tried to ssh directly as Hatter. With the password provided by the Hatter, I now have a more stable SSH connection to the Wonderland machine. Since there was nothing more to find in Hatter's home, I was out of ideas to get root access by hand. So I started a simple Python server on my machine and used 'wget' to download 'linpeas' to the wonderland machine. While scrolling through the 'linpeas' report I came across these lines:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693405097936/092851f2-0666-4a2a-81f2-3bffadc2e548.png align="center")

I searched on [https://gtfobins.github.io/](https://gtfobins.github.io/) for 'capabilities' and was given a possible backdoor for root access.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693405238592/81c88f6d-049f-41f2-a739-a9f1dcec203a.png align="center")

I modified the last command a little bit so it is pointing to the right directory.

```bash
/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

After executing the command I was granted root access, and finally was able to cat the root flag in Alice's home and finish the machine.