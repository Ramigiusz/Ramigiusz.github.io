---
layout: post
title: "TryHackMe 'Summit' Challenge - Walkthrough"
author: "Rami Matouk"
date: 2024-11-08
categories: guides
tags: [piramid of pain]
---

# TryHackMe Summit
Can you chase a simulated adversary up the Pyramid of Pain until they finally back down?

👉 [Summit](https://tryhackme.com/r/room/summit)

Hello! In line with the Pyramid of Pain concept, the objective of this challenge is to simulate an adversary's operational cost. Our goal is to ascend the pyramid and force the attacker to retreat for good. This write-up is intended to document the experience and serve as a learning resource for both myself and the readers. Welcome aboard!

## Scenario
After numerous incident response activities, PicoSecure has decided to strengthen its malware detection capabilities by conducting a threat simulation and detection engineering engagement. You have been tasked to collaborate with an external penetration tester in an iterative purple-team scenario.

The tester's goal is to execute malware samples on a simulated internal user workstation. Meanwhile, your job is to configure PicoSecure's security tools to detect and prevent these malicious activities effectively.

![image](https://github.com/user-attachments/assets/21d4505a-c986-466c-bbf3-9f70076cf42b)


## Pyramid of Pain

![image](https://github.com/user-attachments/assets/7cd26859-98b4-4b03-b894-dcc6c168f31b)

The Pyramid of Pain is a concept that describes various mitigation strategies for attacker methodologies and the associated operational costs for the attacker. Here's a breakdown:
1. **Trivial** - **Hash Values** - Attackers can trivially modify hash values of their malicious software by changing a single bit.
2. **Easy** - **IP Addresses** - Attackers can easily change their IP addresses to evade detection based on static IP rules.
3. **Simple** - **Domain Names** - Changing a domain name is slightly more costly for attackers, as it requires acquiring a new domain, incurring additional expenses.
4. **Annoying** - **Network/Host Artifact**s - Detecting artifacts of an attacker's activity forces them to modify tools or alter methods, increasing their operational burden.
5. **Challenging** - Tools - Successfully detecting and neutralizing an attacker's tools forces them to return to the weaponization phase of the Cyber Kill Chain. They must acquire new tools, undergo further development, or adapt training, creating a significant challenge.
6. **Tough!** - **TTP** (Techniques, Tactics, and Procedures) - Stopping an attacker’s TTP forces them to entirely rethink and plan new strategies for your system. This is often prohibitively resource-intensive, making them abandon their efforts altogether.

## Task 1: Detecting Malware Hash Value
Our first task is to create a detection rule to identify malware execution. We begin by scanning the file:

![image](https://github.com/user-attachments/assets/5d3f7d14-ae0e-42c8-b9bb-d30909219e70)

![image](https://github.com/user-attachments/assets/30650de9-ba3d-4d0f-ad53-a2b7ef778ac3)

As we climb the Pyramid of Pain, the first step is to detect the malware file by its hash value.

![image](https://github.com/user-attachments/assets/cb5b4941-492d-42bb-af6e-d69b3346d1b1)

I chose the SHA256 hash for its robust and secure nature. Unlike MD5, which is susceptible to hash collisions, SHA256 ensures reliable identification.

### Result
After implementing the detection, the penetration tester reported that the execution of `sample1.exe` was successfully prevented. An email confirmation included a flag and another malware sample: `sample2.exe`.

## Task 2: Blocking Malicious IP Addresses
This time our scanning raport is populated with additional network related data.

![image](https://github.com/user-attachments/assets/21e166fd-ab55-45ab-a98b-56867feaa1fa)

The malware attempts to connect to the IP address 154.35.10.113:4444 via the GET method. To stop this behavior, we add a firewall rule:

![image](https://github.com/user-attachments/assets/5cbbab20-3c41-4870-80fe-08677591dfda)

Key Configuration:
- Type: Egress (blocks outgoing connections from our network).
- Source IP: Any (blocks traffic from all internal devices).
- Destination IP: Attacker's IP address (154.35.10.113).
- Action: Deny access.

### Result
This rule successfully prevented connections to the malicious server. The penetration tester confirmed the success via email but remains unfazed for now.

## Task 3
The attacker adapted by switching IP addresses, but the next scan revealed the use of a domain name:

![image](https://github.com/user-attachments/assets/a3a934f4-5c0b-4552-9211-4c83a37144b5)

The malware communicates with emudyn.bresonicz.info. To disrupt this behavior, we created a DNS Filter rule:

![image](https://github.com/user-attachments/assets/8a5a6e36-064d-4d6b-b61e-5a40c92d2510)

Deny all traffic associated with emudyn.bresonicz.info.

### Result
This step significantly increased the attacker's operational cost, as they now need to purchase and register new domain names.

## Task 4
The next scan revealed more sophisticated malware behavior:

![image](https://github.com/user-attachments/assets/37bde0e9-68f5-4d3f-abd5-e1362c8a2d14)

### Observations:
- The malware disables real-time monitoring via registry modifications.
- It downloads backdoor.exe from a new domain.
To combat this, we created a Sigma rule to track registry modifications using Sysmon Events:

![image](https://github.com/user-attachments/assets/2e438e8f-d0c5-432d-8676-69d1d5c5e84a)

![image](https://github.com/user-attachments/assets/16eff868-c92d-4312-8e90-95dc4b3692cb)

Registry values related to real-time monitoring, aligning with MITRE ATT&CK TA0005 - Defense Evasion.

### Result
The attacker is visibly annoyed but continues the attack.
![image](https://github.com/user-attachments/assets/a0e62a9b-5a8b-4d9a-a8db-e7a349227fc9)

## Task 5
The attacker has adapted further, leveraging their C2 (Command and Control) server. A closer analysis reveals:

![image](https://github.com/user-attachments/assets/0e53ce4c-b7c0-4449-a034-fc94d8737407)

### Observations:
- Regular outgoing connections to 51.102.10.19 every 30 minutes.
- Connections are consistently 97 bytes in size.
- The attacker uses port 443.
To detect this, we created a Sysmon network connections rule:

![image](https://github.com/user-attachments/assets/40b15164-dd8e-4426-91c5-7dac9140eec3)

### Result
By targeting these connections, we significantly disrupted the attacker’s ability to control the malware. They are now desperate.

## Task 6
To end the attack, we target the attacker's Techniques, Tactics, and Procedures (TTPs). The attacker relies on a file `exfiltr8.lo` stored in the `%temp%` folder for exfiltration:

![image](https://github.com/user-attachments/assets/53108b3d-0057-4697-9111-65127e9920dd)

We created a Sysmon file creation rule to detect and respond to such attempts:

![image](https://github.com/user-attachments/assets/4623aa44-44ec-4def-a9c0-06b216f3ff40)

This time we will track file creation and modifications rule for sysmon.

### Result
By disrupting the attacker’s TTPs, we forced them to retreat entirely!

![image](https://github.com/user-attachments/assets/921af870-65f6-4396-835a-b0a749225782)

## Congratulations, Defender! You successfully climbed the Pyramid of Pain and neutralized the threat.

