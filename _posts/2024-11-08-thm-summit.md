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

## Task 1 
Our task is to create detection rule that will find execution of malware. We start our job with scanning file

![image](https://github.com/user-attachments/assets/5d3f7d14-ae0e-42c8-b9bb-d30909219e70)

![image](https://github.com/user-attachments/assets/30650de9-ba3d-4d0f-ad53-a2b7ef778ac3)

As we climb pyramid of pain, first way to detect this file (Which is Trojan btw...) we can track it's Hash value.

![image](https://github.com/user-attachments/assets/cb5b4941-492d-42bb-af6e-d69b3346d1b1)

I selected to follow Sha256 hash because of it's secure nature. MD5 for example was proven to have hash collision making it insecure.

After this we receive from penetration tester informing us about successful sample.1exe execution prevention. You will receive e-mail with flag and another malware, sample2.exe.

## Task 2
This time our scanning raport is populated with additional network related data.

![image](https://github.com/user-attachments/assets/21e166fd-ab55-45ab-a98b-56867feaa1fa)

We see that this malware connects to ip address 154.35.10.113:4444 through GET method. Let's add firewall rule to prevent this behaviour.

![image](https://github.com/user-attachments/assets/5cbbab20-3c41-4870-80fe-08677591dfda)

- We select type of direction to "Egress" which means we want to stop all connections from our network to malicious network
- We select source ip as "any". Our goal is to stop connection to this malicious url from any computer
- Destination IP is set to attacker's ip address
- Action is to "Deny" access

After adding this rule we receive another mail from penetration tester informing us about our success! He still stays calm...

## Task 3
This time attacker changed ip address of his server and is equipped with plenty of more. We must block something more painful for him to circumevent. Third step in pyramid of pain is to block Domain Name.

![image](https://github.com/user-attachments/assets/a3a934f4-5c0b-4552-9211-4c83a37144b5)

This time we gain even more information from scanning. We can easily see that attacker uses "emudyn.bresonicz.info" domain name. Let's stop him to deal some damage to his fortitude!
