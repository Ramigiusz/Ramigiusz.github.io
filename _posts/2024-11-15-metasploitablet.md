---
layout: post
title: "Hacking metasploitable 2 machine with well... Metasploit"
author: "Rami Matouk"
date: 2024-10-25
categories: projects
tags: [Metasploit, Red Team]
---


# Attack Preparation

Hello, today we will experiment with powerful Metasploit Framework and Metasploitable 2, made to be vulnerable machine.

## What is Metasploit Framework?
[Metasploit](https://www.metasploit.com/) is a tool created for penetration testing, equipped with powerful array of payloads, exploits and scripts. This tool is used to help Red Team specialist to conduct their attacks.

For this lab i'am using Kali Linux with preinstalled Metasploit and Metasploitable 2 machine which allows us to test this awesome tool in action. These 2 machines are connected together in isolated network. We do not want to expose Metasploitable 2 to internet, this machine is designed to be vulnerable!

![image](https://github.com/user-attachments/assets/14bc7d8c-03a6-4761-b182-4c77c3eb5057)

In my testing environment i refer using IP addresses to those machines:
- 192.168.1.6 - Kali Linux (attacker)
- 192.168.1.7 - Metasploitable 2 (victim)
According to your configuration, these addreses may differ

## Reconessaince with Metasploit
Metasploit is equipped with tools to perform recon of our network and gain more information about our victims.

### Nmap in Metasploit

To begin usage of metasploit we use `sudo msfconsole` command

![image](https://github.com/user-attachments/assets/5fbfe0b5-9061-4c8e-aef7-5a538262b114)

`msf6 >` confirms that metasploit is working.

Now we can scan our network for victims with `nmap`, yes we can use nmap in Metasploit Framework.
```nmap -O <your-network>```

![image](https://github.com/user-attachments/assets/1143a876-dc19-40f0-a8f2-e091a9550a91)

We found machine with 192.168.1.7 address, and we can notice that it's running a lot of services. We can use nmap with flag `-sV` to extract more information about this host

![image](https://github.com/user-attachments/assets/084edb5d-cb68-432b-a36f-0b2b186b2777)

Now we extracted even more information, especially about service version, this is especially useful for finding potential vulnerabilities to exploit.

### Portscan module
You can use `search` function inside framework to search for modules. We are interested in portscan module

![image](https://github.com/user-attachments/assets/5435d8d0-cae9-4f99-8225-8a49b211965d)

We will use **TCP SYN Port Scanner** for our task. To select module write `use [nr]`. After selection use `show options` command to print out settings of this module on terminal. Additionaly if you are interested you can use `info` command to view full module information

![image](https://github.com/user-attachments/assets/fbaab1e9-357a-491f-80e1-3b1b8310a975)

RHOSTS is a name of setting that describes target of this module, we must specify this setting by using `set RHOSTS`. Set command only works in this particular module. It's worth noting that using command `setg` allows us to set parameter for different modules aswell. 

![image](https://github.com/user-attachments/assets/86c52691-096d-4210-ae0c-c9b031aa5804)

Additionaly you can set THREADS to something like 10 to speed up scanning process, at the cost of visibility

Now to run exploit we just write `run` or `exploit`.

![image](https://github.com/user-attachments/assets/cda95b80-b730-489e-b128-18ed0573c174)

From this  command we extract similiar information like with Nmap, we see opened ports but because we are just using SYN scan we are unable to determine type and version of service, we can only guess by number of port.

SYN scan sends only TCP SYN packets, victim answers with SYN/ACK packet and waits for us to send ACK packet to complete TCP connection, however we only send SYN packet to specific port, receive SYN/ACK and flag this port as open

