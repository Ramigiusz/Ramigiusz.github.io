---
layout: post
title: "Nmap for begineers + Cheatsheet"
author: "Rami Matouk"
date: 2024-10-29
categories: guides
---


![image](https://github.com/user-attachments/assets/a1a5edf9-7734-4b6f-bdb2-d2e8b2ba6bb0)

# Nmap Introduction
Imagine a scenario where you need to discover all the various services running on a network, along with their versions and the types of systems hosting these services. Sure, you could perform manual scans to identify live hosts by using `ping` but imagine doing that for 254 devices! And if you want to discover services, you could try `telnet <port-num>`, repeating this over 60,000 times to check every possible port on each device.

This is exactly why we have tools to streamline these tasks. Today, I’ll introduce one of the most powerful tools for this purpose: **Nmap**.

Nmap is an open-source network scanner that has been in development since 1997. It has amassed a vast array of capabilities, making it an indispensable tool for cybersecurity specialists and, unfortunately, a favorite for malicious hackers as well.

# Table of Contents

1. [How to Effectively Discover Live Hosts with Nmap](#1-how-to-effectively-discover-live-hosts-with-nmap)  
   1.1 [Local Network Scan vs. Remote Network Scan](#11-local-network-scan-vs-remote-network-scan)  
   1.2 [Scanning Range Specifications](#12-scanning-ip-range-specifications)  
2. [Port Scanning with Nmap](#2-port-scanning-with-nmap)  
   2.1 [SYN Scan](#21-syn-scan)  
   2.2 [Connect Scan](#22-connect-scan)  
   2.3 [UDP Scan](#23-udp-scan)  
   2.4 [Scanning Port Range Specifications](#24-scanning-port-range-specifications)  
3. [Extracting Additional Information with Nmap](#3-extracting-additional-information-with-nmap)  
4. [Controlling Nmap Scanning Speed](#4-controlling-nmap-scanning-speed)  
5. [When You Need More Detail](#5-when-you-need-more-detail)  
6. [Conclusion](#6-conclusion)
7. [Cheatsheet](#cheatsheet)


## 1. How to Effectively Discover Live Hosts with Nmap?
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

## 3. Extracting Additional Information with Nmap
Nmap can provide much more than just identifying live hosts and open ports. It can reveal further details about systems and services:
- **OS Detection** - Use the `-O` flag to enable OS detection. This scan mode attempts to identify the host's operating system based on various indicators. However, be aware that OS detection isn't foolproof and may occasionally produce incorrect results.

![image](https://github.com/user-attachments/assets/743033e2-a19a-4c00-a76f-a9814a3c3f1b)

As shown here, the scanner was unable to accurately identify the OS version.

- **Service and Version Detection** - The `-sV` flag enables Nmap to detect the version of a running service, making it highly valuable for attackers seeking potential vulnerabilities in specific software versions.

![image](https://github.com/user-attachments/assets/fca9fdf0-c1b4-4e85-9ac0-2ec036b48201)

You can combine OS detection and version scanning with the `-A` flag.

## 4. Controlling Nmap Scanning Speed
Nmap offers control over scan speed, which is critical for penetration testers and hackers, as faster scans are more likely to trigger security defenses like IPS/IDS. Nmap provides several scan timing options:
- `T0` - paranoid
- `T1` - sneaky
- `T2` - polite
- `T3` - normal
- `T4` - aggressive
- `T5` - insane
For instance, `-T0` makes Nmap wait 5 minutes between each port probe, while `-T2` only pauses for 0.4 seconds.

You can also adjust scan speed with parallel service probes, using `--min-parallelism <num>` and `--max-parallelism <num>`. These settings define the minimum and maximum number of probes sent in parallel. By default, Nmap adjusts this automatically, but for stable connections, it can run hundreds of probes simultaneously.

Alternatively, the `--min-rate <num>` and `--max-rate <num>` flags set a global packet rate for the entire scan. Finally, the `--host-timeout <time>` option specifies the maximum time to wait for a target host before moving on, helpful in cases of poor connectivity.



## 5. When You Need More Detail
If a scan is too slow or you just need more information, Nmap offers options for real-time updates:
- `-v` - Verbose mode, which provides real-time updates. You can increase verbosity by adding more vs, such as -vv or up to -v4, for greater detail.
- `-d` - Debug mode, which gives in-depth information for each scan step. Maximum debug level -d9 will produce extensive output, which can flood the screen with details.

In many cases we want to save our scan results. We have several output options:
- `-oN <filename>` - Normal output
- `-oX <filename>` - XML output
- `-oG <filename>` - Output formatted for easy parsing with `grep` or `awk`
- `-oA <filename>` - Output in all major formats

![image](https://github.com/user-attachments/assets/ea4b9db4-c044-4882-9ed0-ce4ab0d6b213)

## 6. Conclusion
For the best experience with Nmap, it’s recommended to run it with elevated privileges. Many of Nmap’s powerful features require administrative access; for instance, the `-sS` (SYN scan) option is only available to users with root or sudo privileges, as sending TCP SYN packets demands higher permissions. Without these privileges, a regular user is limited to `-sT` (TCP connect scan), which completes the full three-way handshake.

I hope this guide helps you get started with Nmap and explore its capabilities. This is my first post on Nmap, and I look forward to sharing more as I continue to learn about this versatile tool!

# Cheatsheet
| **Option**                           | **Explanation**                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------|
| **-sL**                              | List scan – lists targets without scanning                                                       |
| **Host Discovery**                   |                                                                                                  |
| -sn                                  | Ping scan – host discovery only                                                                  |
| **Port Scanning**                    |                                                                                                  |
| -sT                                  | TCP connect scan – completes the three-way handshake                                             |
| -sS                                  | TCP SYN scan – only the first step of the three-way handshake                                    |
| -sU                                  | UDP scan                                                                                         |
| -F                                   | Fast mode – scans the 100 most common ports                                                      |
| -p[range]                            | Specifies a range of port numbers (e.g., -p10-1024) – `-p-` scans all ports                      |
| -Pn                                  | Treats all hosts as online – scans hosts that appear to be down                                  |
| **Service Detection**                |                                                                                                  |
| -O                                   | OS detection                                                                                     |
| -sV                                  | Service version detection                                                                        |
| -A                                   | OS detection, version detection, and other advanced features                                     |
| **Timing**                           |                                                                                                  |
| -T<0-5>                              | Timing template – paranoid (0), sneaky (1), polite (2), normal (3), aggressive (4), insane (5)   |
| --min-parallelism <numprobes>        | Minimum number of parallel probes                                                                |
| --max-parallelism <numprobes>        | Maximum number of parallel probes                                                                |
| --min-rate <number>                  | Minimum rate (packets/second)                                                                    |
| --max-rate <number>                  | Maximum rate (packets/second)                                                                    |
| --host-timeout                       | Maximum time to wait for a target host                                                           |
| **Real-time Output**                 |                                                                                                  |
| -v                                   | Verbosity level – e.g., `-vv` and `-v4`                                                          |
| -d                                   | Debugging level – e.g., `-d` and `-d9`                                                           |
| **Report**                           |                                                                                                  |
| -oN <filename>                       | Normal output                                                                                    |
| -oX <filename>                       | XML output                                                                                       |
| -oG <filename>                       | Grep-able output                                                                                 |
| -oA <basename>                       | Output in all major formats                                                                      |


