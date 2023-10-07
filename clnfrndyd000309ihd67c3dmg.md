---
title: "Vulnhub Matrix 1 Writeup"
seoTitle: "Decoding VulnHub CTF Matrix"
datePublished: Sat Oct 07 2023 08:21:13 GMT+0000 (Coordinated Universal Time)
cuid: clnfrndyd000309ihd67c3dmg
slug: vulnhub-matrix-1-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696666216208/8a0ce7c7-0aff-479f-bd2a-0f4ea94ae683.jpeg
tags: python, ctf, cybersecurity-1, enumeration, vulnhub

---

This is the first of three Matrix-themed CTFs on vulnhub.com. The Description says it is an intermediate CTF, the goal is to get root and read /root/flag.txt. The hint tells us "Follow your intuitions ... and enumerate!".

### Enumeration

Like always

```bash
nmap -sP 10.38.1.0/24
```

is showing us the devices in the target network. So the target machine IP is revealed as 10.38.1.112.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696602038800/de1ebfb6-6a69-403a-b937-f56b2d4e6e71.png align="center")

Let's enumerate a bit further and see what's running on the machine.

```bash
namp -sC -sV 10.38.1.112
```

is showing that we have an open SSH port on port 22, a simple Python HTTP server on port 80 and a simple Python http server on port 31337.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696602156003/110263d0-78d2-42d3-a520-a1cd14a4a4e6.png align="center")

### Initial Access

I decided to investigate the simple Python server on port 80 first.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696661609763/0bf5479d-0eea-4352-9b58-92cc652211af.png align="center")

There\`s a website running telling me to follow the rabbit. It gave me a little flashback from the [Wonderland](https://christophbehr.hashnode.dev/tryhackme-wonderland-writeup) machine on TryHackMe. The clock thing seems to not working. However, I found a small white rabbit icon at the bottom of the site. Clicking it gave me a picture of a white rabbit named "p0rt\_31337"

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696662210812/ea5bc27f-61e8-4074-88db-119db2f31e24.png align="center")

I checked the source of the page but there was nothing more to find. Since there is nothing more I could find on that webpage I switched over to the website on port 31337.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696662295771/2dbc0622-3d2a-44aa-8c36-75d94ed866cd.png align="center")

It's the same page as the one on port 80 the clock seems not to work either. I checked the source of the page. And found an encoded line of text.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696662435107/ffb897e7-d956-4d7b-b73b-ad3826df0dbf.png align="center")

I copied it and put it on [CyberChef](https://cyberchef.org/). Cyberchef is pretty good with "guessing" what encoding is used in smaller lines of text. So I didn't even bother to think about what encoding might be used. And let the CyberChef do its magic and decide the right recipe.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696662799803/4759aca7-ab65-4122-9c81-7fa5d0e768f0.png align="center")

It is another quote from the Matrix movie plus " &gt; Cypher.matrix". I interpreted "&gt;" to look at Cypher.matrix. Visiting 10.38.1.112:31337/Cypher.matrix downloads a .txt file with some very weird code.

```plaintext
 
+++++ ++++[ ->+++ +++++ +<]>+ +++++ ++.<+ +++[- >++++ <]>++ ++++. +++++
+.<++ +++++ ++[-> ----- ----< ]>--- -.<++ +++++ +[->+ +++++ ++<]> +++.-
-.<++ +[->+ ++<]> ++++. <++++ ++++[ ->--- ----- <]>-- ----- ----- --.<+
+++++ ++[-> +++++ +++<] >++++ +.+++ +++++ +.+++ +++.< +++[- >---< ]>---
---.< +++[- >+++< ]>+++ +.<++ +++++ ++[-> ----- ----< ]>-.< +++++ +++[-
>++++ ++++< ]>+++ +++++ +.+++ ++.++ ++++. ----- .<+++ +++++ [->-- -----
-<]>- ----- ----- ----. <++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ +.<++
+[->- --<]> ---.< ++++[ ->+++ +<]>+ ++.-- .---- ----- .<+++ [->++ +<]>+
+++++ .<+++ +++++ +[->- ----- ---<] >---- ---.< +++++ +++[- >++++ ++++<
]>+.< ++++[ ->+++ +<]>+ +.<++ +++++ ++[-> ----- ----< ]>--. <++++ ++++[
->+++ +++++ <]>++ +++++ .<+++ [->++ +<]>+ ++++. <++++ [->-- --<]> .<+++
[->++ +<]>+ ++++. +.<++ +++++ +[->- ----- --<]> ----- ---.< +++[- >---<
]>--- .<+++ +++++ +[->+ +++++ +++<] >++++ ++.<+ ++[-> ---<] >---- -.<++
+[->+ ++<]> ++.<+ ++[-> ---<] >---. <++++ ++++[ ->--- ----- <]>-- -----
-.<++ +++++ +[->+ +++++ ++<]> +++++ +++++ +++++ +.<++ +[->- --<]> -----
-.<++ ++[-> ++++< ]>++. .++++ .---- ----. +++.< +++[- >---< ]>--- --.<+
+++++ ++[-> ----- ---<] >---- .<+++ +++++ [->++ +++++ +<]>+ +++++ +++++
.<+++ ++++[ ->--- ----< ]>--- ----- -.<++ +++++ [->++ +++++ <]>++ +++++
+++.. <++++ +++[- >---- ---<] >---- ----- --.<+ +++++ ++[-> +++++ +++<]
>++.< +++++ [->-- ---<] >-..< +++++ +++[- >---- ----< ]>--- ----- ---.-
--.<+ +++++ ++[-> +++++ +++<] >++++ .<+++ ++[-> +++++ <]>++ +++++ +.+++
++.<+ ++[-> ---<] >---- --.<+ +++++ [->-- ----< ]>--- ----. <++++ +[->-
----< ]>-.< +++++ [->++ +++<] >++++ ++++. <++++ +[->+ ++++< ]>+++ +++++
+.<++ ++[-> ++++< ]>+.+ .<+++ +[->- ---<] >---- .<+++ [->++ +<]>+ +..<+
++[-> +++<] >++++ .<+++ +++++ [->-- ----- -<]>- ----- ----- --.<+ ++[->
---<] >---. <++++ ++[-> +++++ +<]>+ ++++. <++++ ++[-> ----- -<]>- ----.
<++++ ++++[ ->+++ +++++ <]>++ ++++. +++++ ++++. +++.< +++[- >---< ]>--.
--.<+ ++[-> +++<] >++++ ++.<+ +++++ +++[- >---- ----- <]>-- -.<++ +++++
+[->+ +++++ ++<]> +++++ +++++ ++.<+ ++[-> ---<] >--.< ++++[ ->+++ +<]>+
+.+.< +++++ ++++[ ->--- ----- -<]>- --.<+ +++++ +++[- >++++ +++++ <]>++
+.+++ .---- ----. <++++ ++++[ ->--- ----- <]>-- ----- ----- ---.< +++++
+++[- >++++ ++++< ]>+++ .++++ +.--- ----. <++++ [->++ ++<]> +.<++ ++[->
----< ]>-.+ +.<++ ++[-> ++++< ]>+.< +++[- >---< ]>--- ---.< +++[- >+++<
]>+++ +.+.< +++++ ++++[ ->--- ----- -<]>- -.<++ +++++ ++[-> +++++ ++++<
]>++. ----. <++++ ++++[ ->--- ----- <]>-- ----- ----- ---.< +++++ +[->+
+++++ <]>++ +++.< +++++ +[->- ----- <]>-- ---.< +++++ +++[- >++++ ++++<
]>+++ +++++ .---- ---.< ++++[ ->+++ +<]>+ ++++. <++++ [->-- --<]> -.<++
+++++ +[->- ----- --<]> ----- .<+++ +++++ +[->+ +++++ +++<] >+.<+ ++[->
---<] >---- .<+++ [->++ +<]>+ +.--- -.<++ +[->- --<]> --.++ .++.- .<+++
+++++ [->-- ----- -<]>- ---.< +++++ ++++[ ->+++ +++++ +<]>+ +++++ .<+++
[->-- -<]>- ----. <+++[ ->+++ <]>++ .<+++ [->-- -<]>- --.<+ +++++ ++[->
----- ---<] >---- ----. <++++ +++[- >++++ +++<] >++++ +++.. <++++ +++[-
>---- ---<] >---- ---.< +++++ ++++[ ->+++ +++++ +<]>+ ++.-- .++++ +++.<
+++++ ++++[ ->--- ----- -<]>- ----- --.<+ +++++ +++[- >++++ +++++ <]>++
+++++ +.<++ +[->- --<]> -.+++ +++.- --.<+ +++++ +++[- >---- ----- <]>-.
<++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ .++++ +++++ .<+++ +[->- ---<]
>--.+ +++++ ++.<+ +++++ ++[-> ----- ---<] >---- ----- --.<+ +++++ ++[->
+++++ +++<] >+.<+ ++[-> +++<] >++++ .<+++ [->-- -<]>- .<+++ +++++ [->--
----- -<]>- ---.< +++++ +++[- >++++ ++++< ]>+++ +++.+ ++.++ +++.< +++[-
>---< ]>-.< +++++ +++[- >---- ----< ]>--- -.<++ +++++ +[->+ +++++ ++<]>
+++.< +++[- >+++< ]>+++ .+++. .<+++ [->-- -<]>- ---.- -.<++ ++[-> ++++<
]>+.< +++++ ++++[ ->--- ----- -<]>- --.<+ +++++ +++[- >++++ +++++ <]>++
.+.-- .---- ----- .++++ +.--- ----. <++++ ++++[ ->--- ----- <]>-- -----
.<+++ +++++ [->++ +++++ +<]>+ +++++ +++++ ++++. ----- ----. <++++ ++++[
->--- ----- <]>-- ----. <++++ ++++[ ->+++ +++++ <]>++ +++++ +++++ ++++.
<+++[ ->--- <]>-- ----. <++++ [->++ ++<]> ++..+ +++.- ----- --.++ +.<++
+[->- --<]> ----- .<+++ ++++[ ->--- ----< ]>--- --.<+ ++++[ ->--- --<]>
----- ---.- --.<
```

I searched online for hints on what this could be. Quickly I found that this is a language called BrainFuck. With an online Brainfuck decoder I was able to translate the file. It said, "You can enter into matrix as guest, with password k1ll0rXX Note: Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password". These are clear instructions to log in via SSH. I wrote a short Python script to run through all combinations of letters and numbers.

```python
import itertools

password_incomplete = "k1ll0r"
characters = ["a", "b", "c", "d", 
              "e", "f", "g", "h", 
              "i", "j", "k", "l", 
              "m", "n", "o", "p", 
              "q", "r", "s", "t", 
              "u", "v", "w", "x", 
              "y", "z", "1", "2", 
              "3", "4", "5", "6", 
              "7", "8", "9", "0"]

for i in range(len(characters)+1):
    for j in itertools.combinations(characters, i):
        password = "".join(j)
        if len(password) <= 2:
            print(password_incomplete + password)
```

I piped the output of the script to passwords.txt

```bash
python ./passwords.py > passwords.txt
```

And used the created passwords.txt as a list to crack the password for the guest user with Hydra.

```bash
hydra -l guest -P passwords.txt 10.38.1.112 ssh
```

The correct password was quickly cracked and I could log in as the guest user. Unfortunately, it's only a restricted shell.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696665615555/82fd2d2d-19c5-44f5-997c-cd0875f29bb0.png align="center")

There are a lot of possibilities to [bypass](https://0xffsec.com/handbook/shells/restricted-shells/) the restriction and I tried a lot of them. Finally

```bash
sudo ssh guest@10.38.1.112 -t bash
```

gave me access to an unrestricted shell.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696665826681/8d0ec162-f8f1-433b-99b7-e9d35532d0ab.png align="center")

### Root Access

I used SCP and the Python simple HTTP server to download linpeas on the target machine. After it finished I scrolled through linpeas' output.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696665898674/dab1e01a-633a-418c-9f10-c73d5fd1f31b.png align="center")

I gave myself a facepalm because I could've found that out by simply typing

```bash
sudo -l
```

in my shell. But here we are... The guest user is in the ALL group. This means with a simple

```bash
sudo su
```

I'm granted root access, hence I was able to get the root flag and complete the machine.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696666258446/0de44e7e-7df2-45ca-ada9-bdcdffe8e603.png align="center")