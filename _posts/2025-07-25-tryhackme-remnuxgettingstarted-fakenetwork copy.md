---
title: "TryHackMe: REMnux Getting Started - Task 4 Fake Network to Aid Analysis"
author: Dustin Merritt
categories: [TryHackMe]
tags: [remnux, inetsim, dynamic analysis, malware, network, simulation]
render_with_liquid: false
media_subpath: /images/tryhackme_remuxgettingstarted/
image:
  path: room_image.webp
---

**INetSim** was used on the **REMnux** VM to simulate a fake network for observing the behavior of potentially malicious software. This method is useful in dynamic analysis, especially when malware attempts to establish network connections or download secondary payloads.

[TryHackMe Room: REMnux Getting Started](https://tryhackme.com/room/remnuxgettingstarted)


## Configuring INetSim

First, we retrieved the IP address of the REMnux VM using `ifconfig` or by checking the prompt:

```bash
ubuntu@10.10.23.89:~$
```

Then we edited the configuration file:

```bash
sudo nano /etc/inetsim/inetsim.conf
```

We found the line:

```ini
#dns_default_ip  0.0.0.0
```

And changed it to:

```ini
dns_default_ip 10.10.23.89
```

We confirmed it with:

```bash
cat /etc/inetsim/inetsim.conf | grep dns_default_ip
```

Finally, we started INetSim:

```bash
sudo inetsim
```

> Ensure you see “Simulation running” at the bottom to confirm it's working. You can ignore the "http_80_tcp - failed!" message.

## Simulating Malware Behavior

Switch to the **AttackBox** and open the browser to visit:

```
https://10.10.23.89
```

Accept the self-signed certificate risk.

Next, simulate malware activity by downloading files via the CLI:

```bash
sudo wget https://10.10.23.89/second_payload.zip --no-check-certificate
sudo wget https://10.10.23.89/second_payload.ps1 --no-check-certificate
```

These files will be saved in the `/root` folder. Open the `.ps1` file — it simply points back to INetSim’s homepage.

> This demonstrates typical behavior of droppers: reaching a remote server to pull in an additional payload.

## Reviewing Connection Logs

When finished, stop INetSim (Ctrl + C), and it will write a report to:

```bash
/var/log/inetsim/report/
```

Read the report with:

```bash
sudo cat /var/log/inetsim/report/report.XXXX.txt
```

Sample output:

```
2024-09-22 21:18:37  HTTPS connection, method: GET, URL: https://10.10.23.89/second_payload.ps1
2024-09-22 21:18:49  HTTPS connection, method: GET, URL: https://10.10.23.89/second_payload.zip
2024-09-22 21:20:00  HTTPS connection, method: GET, URL: https://10.10.23.89/flag.txt
```

## Final Step: Download the Flag

Use this command to grab the flag:

```bash
sudo wget https://10.10.23.89/flag.txt --no-check-certificate
```

---

<style>
.center img 
  display:block;
  margin-left:auto;
  margin-right:auto;
}
.wrap pre{
    white-space: pre-wrap;
}
</style>
