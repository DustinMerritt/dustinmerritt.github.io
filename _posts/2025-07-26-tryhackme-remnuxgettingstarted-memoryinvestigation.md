---
title: "TryHackMe: REMnux Getting Started - Task 5 Memory Investigatoin: Evidence Preprocessing"
author: Dustin Merritt
categories: [TryHackMe]
tags: [volatility, remnux, memory forensics, evidence preprocessing]
render_with_liquid: false
media_subpath: /images/tryhackme_remuxgettingstarted/
image:
  path: room_image.webp
---

Task 5 focuses on memory investigation and evidence preprocessing. This was my first hands-on experience with the Volatility 3 framework, and I want to share what I’ve learned in case it helps someone else getting started.

---

## What Is Evidence Preprocessing?

One of the first things I’ve learned is that preprocessing means running tools over captured memory images to extract important artifacts before diving deep into analysis. These artifacts can include running processes, command-line arguments, injected code, and more.

A common tool for this is Volatility, which is already included in the REMnux VM. REMnux is a Linux distro focused on reverse engineering and malware analysis, and I found it pretty intuitive after getting used to the CLI workflow.

---

## Preprocessing with Volatility 3

For this task, I used Volatility 3, the newer version of the framework. The focus wasn’t to analyze the results in depth yet, but just to get familiar with the tool and understand what kind of information each plugin can extract.

Each plugin can take 2–3 minutes to run depending on system performance.

### Windows Plugins I Ran

Here are the three Windows-specific plugins I experimented with:

- windows.pstree.PsTree  
  Shows a hierarchical view of processes, which helped me understand how processes relate to each other—like seeing a suspicious child process launched from explorer.exe.

- windows.pslist.PsList  
  Gives a flat list of all active processes from the memory image. A good overview of what was running.

- windows.cmdline.CmdLine  
  Shows how processes were launched and with what arguments. This was helpful in spotting processes launched with strange parameters.

---

## Running the Plugins

Here’s how I ran the plugins individually:

```bash
volatility3 -f memory.img windows.pslist.PsList
```

Then I decided to automate it with a short Bash script:

```bash
#!/bin/bash
for plugin in windows.pslist.PsList windows.pstree.PsTree windows.cmdline.CmdLine; do
  volatility3 -f memory.img $plugin > output_$plugin.txt
done
```

Tip: Store the output in folders named after your case or investigation. For example: cases/REMnux_Task5/output/

---

Once the outputs were saved, I had a much clearer picture of what the system looked like at the time the memory was captured. This forms a solid base for deeper analysis like malware tracing, persistence hunting, or suspicious activity detection.

---

## Questions & Answers

Q1: What plugin lists processes in a tree based on their parent process ID?  
A: PsTree

Q2: What plugin is used to list all currently active processes in the machine?  
A: PsList

Q3: What Linux utility tool can extract the ASCII, 16-bit little-endian, and 16-bit big-endian strings?  
A: strings

Q4: By running vol3 with the Malfind parameter, what is the first (1st) process identified suspected of having an injected code?
Command Used:
```
vol3 -f wcry.mem windows.malfind.Malfind
```
A: csrss.exe

Q5: What is the second (2nd) process identified suspected of having an injected code?  
A: winlogon.exe

Q6: By running vol3 with the DllList parameter, what is the file path or directory of the binary @WanaDecryptor@.exe?
Command Used:
```
vol3 -f wcry.mem windows.dlllist.DllList | grep "WanaDecryptor"
```
A: C:\Intel\ivecuqmanpnirkt615

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
