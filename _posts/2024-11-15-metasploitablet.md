---
layout: post
title: "Hacking Metasploitable 2 Machine with Metasploit"
author: "Rami Matouk"
date: 2024-11-15
categories: guides
tags: [Metasploit, Red Team]
---


# Metasploit in action

Hello! Today, we will experiment with the powerful Metasploit Framework and Metasploitable 2, a deliberately vulnerable virtual machine.

## What is Metasploit Framework?
[Metasploit](https://www.metasploit.com/) is a tool created for penetration testing, equipped with an extensive array of payloads, exploits, and scripts. It is widely used by Red Team specialists to simulate attacks and assess the security of systems

For this lab, I am using Kali Linux with a pre-installed Metasploit Framework and a [Metasploitable-2](https://docs.rapid7.com/metasploit/metasploitable-2/) machine to test this tool in action. These two machines are connected within an isolated network. We do not want to expose Metasploitable 2 to the internet, as it is designed to be vulnerable!

![image](https://github.com/user-attachments/assets/14bc7d8c-03a6-4761-b182-4c77c3eb5057)

In my testing environment, I use the following IP addresses:
- 192.168.1.6 - Kali Linux (attacker)
- 192.168.1.7 - Metasploitable 2 (victim)
Note: Your configuration might use different IP addresses.

## Reconnaissance with Metasploit
Metasploit has built-in tools for network reconnaissance, helping gather information about potential targets.

### Using Nmap in Metasploit

To start Metasploit, use the `sudo msfconsole` command:

![image](https://github.com/user-attachments/assets/5fbfe0b5-9061-4c8e-aef7-5a538262b114)

The prompt `msf6 >` confirms that Metasploit is running.

To scan the network, we can use nmap directly within Metasploit:
```nmap -O <your-network>```

![image](https://github.com/user-attachments/assets/1143a876-dc19-40f0-a8f2-e091a9550a91)

We discover a machine with the IP address 192.168.1.7, running multiple services. To get more detailed information, use the `-sV` flag:

![image](https://github.com/user-attachments/assets/084edb5d-cb68-432b-a36f-0b2b186b2777)

This provides valuable information about the versions of services running on the target, which is essential for identifying vulnerabilities.

### Portscan module
To find relevant modules in Metasploit, use the `search` function. For example, to find port scanning tools:

![image](https://github.com/user-attachments/assets/5435d8d0-cae9-4f99-8225-8a49b211965d)

For this task, we’ll use the TCP SYN Port Scanner module. Select it by typing `use [module_number]`, then use `show options` to see its configuration. The info command provides full module details.

![image](https://github.com/user-attachments/assets/fbaab1e9-357a-491f-80e1-3b1b8310a975)

Set the RHOSTS parameter (target IP) with 
```set RHOSTS <target_ip>```
Note: `setg` can be used to set parameters globally across multiple modules.

![image](https://github.com/user-attachments/assets/86c52691-096d-4210-ae0c-c9b031aa5804)

Optionally, adjust `THREADS` to speed up scanning at the cost of stealth.
Run the scan with `run` or `exploit`:

![image](https://github.com/user-attachments/assets/cda95b80-b730-489e-b128-18ed0573c174)

This provides similar results to nmap, identifying open ports. Note that with a SYN scan, only TCP SYN packets are sent. The victim responds with a SYN/ACK, signaling an open port without fully establishing a connection.

## How to perform a Basic Attack
To conduct a basic attack, you need to know:
1. The service version.
2. The vulnerabilities associated with that version.
This process is a common starting point for attackers who do not develop new exploits but use existing ones. We can find vulnerabilites [here](https://www.exploit-db.com/)

### Version scanning
Start by searching for a module that can identify service versions. For example, to target an FTP server `search ftp version scanner`

![image](https://github.com/user-attachments/assets/b0b34907-8967-49a7-8319-aaef20900138)

![image](https://github.com/user-attachments/assets/35fb4854-a399-481e-a9af-80cabee4c9b1)

We see that the target is running vsFTPd 2.3.4. Next, we search for any known vulnerabilities related to this version

### Attacking FTp server
Let’s take this further and demonstrate an attack on the vulnerable vsFTPd 2.3.4 service. This exploit provides us with a shell inside the vulnerable machine, enabling deeper access and control. Search Metasploit for an exploit:

![image](https://github.com/user-attachments/assets/26f3d679-1598-49a6-a133-8d8c32939e86)

![image](https://github.com/user-attachments/assets/c6e4d09a-5406-471e-b36f-6c4e6942750a)

Configure the necessary parameters and launch the exploit:

![image](https://github.com/user-attachments/assets/9830ce16-c0ff-4367-bb70-51d811e7d07f)

Success! I was able to read a secret message on the target machine.


Feel free to explore and experiment with other services on Metasploitable 2—many of them have known vulnerabilities. Happy hacking and stay safe!
