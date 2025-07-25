---
title: 'TryHackMe: Stolen Mount'
author: Dustin Merritt
categories: [TryHackMe]
tags: [nfs, wireshark, pcap, forensics]
render_with_liquid: false
media_subpath: /images/tryhackme_stolenmount/
image:
  path: room_card.webp
---  
An intruder has breached the internal network and targeted the NFS (Network File System) server where sensitive backup files are stored. A confidential secret was exfiltrated. The only trace left behind is a packet capture (`challenge.pcapng`). Our task is to analyze this PCAP and retrieve the stolen information.

[TryHackMe: Stolen Mount](https://tryhackme.com/room/hfb1stolenmount)

## Tools Used
- Wireshark – To analyze network traffic
- CrackStation.net – To crack the MD5 hash
- imagetotext.info/qr-code-scanner – To extract the contents of the final QR code

## Step-by-Step Walkthrough

### 1. Open the Packet Capture
Opened the `challenge.pcapng` file using Wireshark.

```bash
wireshark ~/Desktop/challenge.pcapng
```

### 2. Filter NFS Traffic
Filtered the traffic by NFS (Network File System):

![NFS Filter](wireshark_nfsfilter.webp){: width="700" }

NFS is used to mount directories over the network and can expose sensitive data if misconfigured.

### 3. Follow TCP Stream
Browsed the filtered traffic and found a stream containing an MD5 hash and a label suggesting it's an archive password:

```
Archive Password: 90eb7723a657b6597100aafef171d9f2 (md5)
```

![TCP Stream with MD5](wireshark_tcpstream.webp){: width="700" }

### 4. Look for File Reads
Searched for `ACCESS : Allowed` and `READ_PLUS` operations to find full file reads which can indicate file downloads over NFS.

![Access Allowed Screenshot](wireshark_accessallowed.webp){: width="700" }

### 5. Export File from PCAP
Identified a file transfer in the traffic. Exported packet bytes as `secrets.zip` using:

- Right-click on packet
- Follow -> TCP Stream
- Save raw bytes as `secrets.zip`

![Exported Bytes](wireshark_exportbytes.webp){: width="700" }

### 6. Crack the Password
The zip file was password protected. Used [https://crackstation.net](https://crackstation.net) to crack the MD5 hash:

![CrackStation MD5 Result](crackstation_md5password.webp){: width="700" }

CrackStation revealed the password successfully.

### 7. Extract the Zip File
Extracted contents of `secrets.zip` using the cracked password:

![Unzip Result](unzip_secrets.txt.webp){: width="700" }

This extracted a QR code image.

![QR Code from Archive](qrcode_secretspng.webp){: width="700" }

### 8. Decode the
 QR Code
Uploaded the QR code image to [https://imagetotext.info/qr-code-scanner](https://imagetotext.info/qr-code-scanner) to decode it. The decoded output revealed the final flag.

![Decoded QR Flag](qrcodereader_flag.webp){: width="700" }

## Flag

```
THM{REDACTED_FOR_WRITEUP}
```

## Lessons Learned

- NFS traffic can expose sensitive files if not secured properly.
- PCAPs are valuable forensic artifacts for breach investigation.
- Weak hashes like MD5 can easily be cracked with online tools.
- Wireshark can reconstruct transferred files from captured packets.

