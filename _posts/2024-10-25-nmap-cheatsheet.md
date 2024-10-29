---
layout: post
title: "Nmap for begineers + Cheatsheet"
author: "Rami Matouk"
date: 2024-10-29
categories: cheatsheets
---


![image](https://github.com/user-attachments/assets/a1a5edf9-7734-4b6f-bdb2-d2e8b2ba6bb0)

# Nmap Introduction
Imagine a scenario where you need to discover all the various services running on a network, along with their versions and the types of systems hosting these services. Sure, you could perform manual scans to identify live hosts by using `ping` but imagine doing that for 254 devices! And if you want to discover services, you could try `telnet <port-num>`, repeating this over 60,000 times to check every possible port on each device.

This is exactly why we have tools to streamline these tasks. Today, I’ll introduce one of the most powerful tools for this purpose: **Nmap**.

Nmap is an open-source network scanner that has been in development since 1997. It has amassed a vast array of capabilities, making it an indispensable tool for cybersecurity specialists and, unfortunately, a favorite for malicious hackers as well.

## 1. How effectively discover live hosts with Nmap? 
Let's start with one of Nmap's basic but highly useful capabilities: discovering live hosts. For this task, Nmap provides the `-sn` option.

![image](https://github.com/user-attachments/assets/47aabce9-3d02-410c-a5dc-3fd030c20e49)

The `-sn` scan is called a "ping scan." Its purpose is to discover live hosts without probing for specific services running on them. This is helpful because it generates minimal network noise—an important consideration, as more detailed scans tend to produce more detectable activity. We’ll explore some of these "noisier" scans in just a moment.

### 1.1 Local Network Scan vs. Remote Network Scan
Imagine a scenario where our device has the IP address `192.168.1.4~`, and we want to scan `192.168.1.31`. In this case, we're scanning a local network.

We define a local network as one where our device is directly connected to the target, such as in Ethernet or WiFi networks. This allows Nmap to utilize the ARP protocol to discover MAC addresses. When scanning local networks, Nmap begins with an ARP request, and if a device responds with an ARP reply, it is marked as a live host.

Now, consider a different scenario where we still use `192.168.1.4` but try to scan `192.168.2.3`. Here, we’re scanning a remote network. A remote network is defined as one where our device is separated from the target by at least one router, which prevents our ARP requests from being broadcast further into the network.

In this situation, Nmap cannot use ARP, as we lack a direct connection to the target device. Instead, Nmap relies on other methods, such as ICMP (ping) or TCP-based scans, to detect live hosts across network boundaries. 

### 1.2 Scanning IP range specifications.
Nmaps supports ways to specify range of the scan.
- **By ip range:** `nmap -sn 192.168.1.1-24` will scan all ip addresses from 192.168.1.1 to 192.168.1.24
- **By subnet:** `nmap -sn 192.168.1.0/24` will scan all ip addresses from 192.168.1.0 to 192.168.1.255
- **By hostname:** `nmap -sn example.com`

Additionally, Nmap provides an option to preview the scan range before executing it, using the -sL (list scan) option. This is useful for verifying the scope of the scan before actually performing it.

## 2. Port scanning with Nmap
Now that we know how to identify which hosts are "up," let's explore how to discover the services running on these devices. By service, we mean any process actively listening for incoming connections on a specific TCP or UDP port.

### 2.1 SYN Scan
The SYN scan, also known as a "stealth" scan, targets TCP ports by sending only a TCP SYN packet. This means the TCP three-way handshake is never completed, resulting in fewer logs and, therefore, reduced network noise. This type of scan is useful when attempting to avoid detection. We can enable a SYN scan with the `-sS` flag. 

![image](https://github.com/user-attachments/assets/76de1dd5-e035-447b-9b13-3cb3965c8b07)

### 2.2 Connect scan
Another scan type is the connect scan, which completes the TCP three-way handshake with each attempted port. If a connection is successfully established, Nmap immediately breaks it and marks the port as open. To perform a connect scan, use the `-sT` flag.

![image](https://github.com/user-attachments/assets/006e3df3-a8da-4b3f-847d-e8193231538e)

### 2.3 UDP scan
The UDP scan is designed to probe UDP ports rather than TCP. While TCP is used by most services, certain critical services rely on UDP, such as DNS, DHCP, NTP, SNMP, and VoIP. UDP is well-suited for real-time communication, making it essential to include in certain scans. Use the `-sU` flag to perform a UDP scan.

### 2.4 Scanning port range specifications.
Nmap allows us to specify the range of ports we want to scan, providing flexibility based on our needs:
- `-F` - Fast mode, which scans the 100 most common ports instead of the default 1,000 ports.
- `-p10-1024` - Specifies a port range. In this example, Nmap will scan ports from 10 to 1,024.
- `-p-` or `-p1-65535` - Scans all ports, from 1 to 65,535.
