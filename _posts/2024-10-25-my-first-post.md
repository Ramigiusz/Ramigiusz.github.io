---
layout: post
title: "How to build SOC Automation Lab at home"
author: "Rami Matouk"
date: 2024-10-25
categories: projects
tags: [SOC, SOAR, SIEM, cybersecurity]
---

# SOC Automation Lab

## Project Introduction
This SOC Automation Lab project is all about setting up a mini Security Operations Center (SOC) at home. Using tools like Wazuh, Shuffle, and TheHive, the goal is to build a SOAR (Security Orchestration, Automation, and Response) system that can automatically detect, analyze, and respond to security threats without needing constant human input. It's a hands-on way to learn how to get different security tools to work together, enrich threat data, and automate responses, making the whole process smoother and more efficient.

![image](https://github.com/user-attachments/assets/33f19026-9578-4ed2-a3c1-7c92076fadf2)

### Skills Learned
- Grasping how SOAR systems work and how to set them up.
- Integrating and automating different security tools.
- Getting better at spotting threats and responding automatically.
- Learning how to enrich security alerts with additional data.
- Building workflows that make security operations quicker and easier.
### Tools Used
- Wazuh for monitoring security events and analyzing data.
- Shuffle for creating automation workflows that handle alerts and incidents.
- TheHive for managing incidents and helping the team collaborate.
- Sysmon for detailed system monitoring on the Windows Wazuh Agent, capturing key event data.

## Table of Contents
- [1. Install Applications & Virtual Machines](#1-install-applications-and-virtual-machines)
  - [1.1 VirtualBox Installation](#11-virtualbox-installation)
  - [1.2 Windows 10 Installation](#12-windows-10-installation)
  - [1.3 Install VirtualBox Guest Additions](#13-install-virtualbox-guest-additions)
  - [1.4 Install Sysmon for Enhanced Event Tracking](#14-install-sysmon-for-enhanced-event-tracking)
  - [1.5 Install and configure Ubuntu VMs](#15-install-and-configure-ubuntu-vms)
- [2. TheHive Configuration](#2-thehive-configuration)
  - [2.1 Cassandra Configuration](#21-cassandra-configuration)
  - [2.2 ElasticSearch Configuration](#22-elasticsearch-configuration)
  - [2.3 TheHive Configuration](#23-thehive-configuration)
- [3. Wazuh Configuration](#3-wazuh-configuration)
  - [3.1 Connect Win10 Machine with Wazuh Manager](#31-connect-win10-machine-with-wazuh-manager)
  - [3.2 Configuring Sysmon Log Ingestion for Wazuh](#32-configuring-sysmon-log-ingestion-for-wazuh)
- [4. Mimikatz Test](#4-mimikatz-test)
- [5. Modify Wazuh Configuration](#5-modify-wazuh-configuration)
  - [5.1 Log all events](#51-log-all-events)
  - [5.2 Custom Wazuh Rule Creation](#52-custom-wazuh-rule-creation)
- [6. Shuffle Configuration](#6-shuffle-configuration)
- [7. Building Workflow](#7-building-workflow)
  - [7.1 Extracting Hash Value](#71-extracting-hash-value)
  - [7.2 Check Reputation Score with VirusTotal](#72-check-reputation-score-with-virustotal)
  - [7.3 Send Details to TheHive](#73-send-details-to-thehive)
  - [7.4 Send Email to the Analyst](#74-send-email-to-the-analyst)
- [8. Bonus: Stoping SSH Bruteforce Attempts](#8-stopping-ssh-bruteforce-attempts)
- [Sources](#sources)
## Steps

### 1. Install Applications and Virtual Machines
We'll kick off our project by setting up the necessary virtual machines and tools in our lab environment. This step involves the installation, configuration, and implementation of the key components we'll be using.

By the end of this step, we will have:
- A Windows 10 machine with Sysmon installed.
- A Wazuh Server.
- A TheHive Server.

#### 1.1 VirtualBox Installation
For virtualization, we will use VirtualBox software, which will provide us with the environment needed to deploy virtual machines. You can download and install VirtualBox from their official website https://www.virtualbox.org/wiki/Downloads

During installation, it's recommended to compare the checksums to verify the integrity of the downloaded file. You can find more information about this on the VirtualBox downloads page.

#### 1.2 Windows 10 Installation
Start by downloading the Windows 10 installation media from the official Microsoft website https://www.microsoft.com/en-us/software-download/windows10 Then follow these steps:
1. Choose the option to create installation media for a different computer.
2. Select the option to download an ISO file.
3. Choose your desired location and complete the download.

In VirtualBox, click on "New" to create a new virtual machine. When prompted, select the Windows 10 ISO file you downloaded earlier.

![image](https://github.com/user-attachments/assets/eec4140e-a27c-4d9c-bb4a-62e259965da9)


I recommend selecting "Unattended Installation" if you want to manually install the OS, but you can choose to do this differently if you prefer. Next, you'll be asked to configure the hardware specifications for your virtual machine. Here's a recommended setup:
- At least 4 GB of RAM.
- 2 CPUs.
- 50 GB of storage.
This configuration should ensure that the machine runs smoothly, but consider your host PC capabilities.

![image](https://github.com/user-attachments/assets/2f96a6c8-d1ed-41ea-984f-91790db64f3a)

Once configured, you should see your machine listed in the VirtualBox dashboard.

Launch your virtual machine and proceed with the Windows installation. During the process, when asked for a product key, select "I don't have a product key." Then, choose "Windows 10 Pro" as the version to install. Select "Custom: Install Windows only" to continue. Windows will now begin installing in the background.

![image](https://github.com/user-attachments/assets/bac583e8-8231-449d-b432-63f40a5c3768)

#### 1.3 Install VirtualBox Guest Additions
To enhance your Windows 10 virtual machine's performance and enable features like shared folders, improved video support, and seamless mouse integration, we'll install VirtualBox Guest Additions. Here’s how to do it:
1. With your Windows 10 virtual machine running, click on "Devices" in the VirtualBox menu.
2. Select "Insert Guest Additions CD Image" from the dropdown menu.
3. Open "This PC" in Windows Explorer, where you will see the Guest Additions CD drive.
4. Double-click the drive to open it, then run the installation file (VBoxWindowsAdditions.exe).
5. Follow the prompts to complete the installation.
Once the installation is finished, restart your virtual machine to apply the changes.

![image](https://github.com/user-attachments/assets/4bb97b74-a28a-4868-8ad1-c9306798d99a)
![image](https://github.com/user-attachments/assets/aed61c5d-0d81-4e0d-b346-84ee7ccf2cc5)

#### 1.4 Install Sysmon for Enhanced Event Tracking
The final step in setting up your Windows 10 machine is to install Sysmon from Sysinternals. Sysmon is a powerful tool that significantly enhances your event tracking capabilities, making it invaluable for monitoring system activity and detecting potential threats. This will be crucial for the practical examples and use cases we'll explore in this project.

Here’s how to get Sysmon up and running:

1. Download Sysmon:
    Visit the official Sysmon download page on Microsoft’s website. https://learn.microsoft.com/pl-pl/sysinternals/downloads/sysmon
    Download the latest version of Sysmon to your Windows 10 virtual machine. 
2. Obtain a Configuration File:
    - To configure Sysmon effectively, you'll need a well-balanced configuration file. A great resource for this is the Sysmon Modular project on GitHub.
    - Go to Sysmon Modular on GitHub. https://github.com/olafhartong/sysmon-modular
    - Look for the sysmonconfig.xml file. This file offers a balanced configuration for Sysmon, providing a good mix of detailed event logging without overwhelming your system with excessive data.
   
    ![image](https://github.com/user-attachments/assets/16b1cfee-523c-4266-86b2-10619b559220)
    ![image](https://github.com/user-attachments/assets/b2a28cd9-07ac-4241-8b3e-3914bd9d0f1c)

3. Install Sysmon with the Configuration:
    - Once you have both Sysmon and the sysmonconfig.xml file, open a command prompt with administrative privileges.
    - Navigate to the directory where you downloaded Sysmon.
    - Run the following command to install Sysmon with your chosen configuration file:
    ```
    sysmon -accepteula -i sysmonconfig.xml
    ```
    This command installs Sysmon, accepts the license agreement, and applies the balanced configuration from the sysmonconfig.xml file.
   ![image](https://github.com/user-attachments/assets/2b139299-45ad-4381-a941-d651892042a9)

5. Restart your virtual machine to ensure that Sysmon starts logging events
6. Check if Sysmon is running:

   You can easily check is Sysmon is running by examining event logs itself, just head to the Event Viewer and find Sysmon logs.
   **Applications and Services -> Microsoft -> Windows -> Sysmon**
    ![image](https://github.com/user-attachments/assets/ddb26fe8-144b-4edd-89fe-8f5960f9afda)

#### 1.5 Install and configure Ubuntu VMs
To complete this step, we'll install Ubuntu systems for our Wazuh Manager and TheHive. Both TheHive and Wazuh are compatible with Ubuntu 22.04 LTS.

**Wazuh Installation**

![image](https://github.com/user-attachments/assets/1dc71d67-85b4-43de-9342-e061b1903276)

I'll be using DigitalOcean's cloud services for both virtual machines to conserve local computer resources. You can choose any cloud provider or create additional VMs on your local machine. I opted for DigitalOcean because they offer a free $200 credit, which should be sufficient to complete our project without incurring any costs.
Next, we'll set up the Wazuh Manager on a DigitalOcean droplet (DigitalOcean's term for virtual machines). 
Follow these steps:
- Navigate to the DigitalOcean dashboard and click on **Create -> Droplets.**
- Select a region that is **geographically closest** to you for better latency and performance.
- Choose **Ubuntu 22.04 LTS** as the operating system image.
- Select a droplet with **8 GB of RAM** and **50 GB of storage** to ensure it can handle the Wazuh Manager's requirements.
- For security, especially since this machine will be exposed to the internet, choose a **strong authentication method**. I will be using password, but if you prefer, you can select a SSH keys. Ensure the password is strong and unique.
- Give your droplet a meaningful name, such as** "WazuhManager"**, to easily identify it in your dashboard.
- Once you've configured everything, click **Create Droplet** to start the deployment process.

![image](https://github.com/user-attachments/assets/0b8e391a-449c-40a7-9f52-89de3f25de30)

While your droplet is deploying, it's important to secure it by setting up firewall rules. We'll configure the firewall to only allow traffic from your machine to this VM.
Navigate to **Networking** and **Firewalls**

![image](https://github.com/user-attachments/assets/85d0d3bf-d57d-40e9-9b86-0c3f8fd44fbd)

By default, DigitalOcean allows SSH access to your droplet from any machine, but we want to restrict that to ensure your Wazuh Manager droplet is secure. We’ll create inbound firewall rules to block all traffic except from your local computer. These rules will cover both TCP and UDP traffic. For the source IP, use your public IP address, which you can find at [WhatIsMyIP.com.](https://www.whatismyip.com/)

Once you've created the firewall, navigate to the **Droplets** section in your DigitalOcean dashboard, select your **droplet**, go to **Networking**, and scroll down to the **Firewalls** section. Click **Edit**, then select the firewall you just created. Under **Droplets**, click **Add Droplets**, and select your **Wazuh Manager VM**.

Now, the firewall is applied to your droplet, securing it by restricting access to only your machine.

![image](https://github.com/user-attachments/assets/9a3a99aa-96d2-4a9e-8d9b-9889c77ac0de)

You can access the droplet through the DigitalOcean web interface. To do this, go to your droplet's page, select **Access** from the menu, and then click **Launch Console**.

Personally, I encountered an issue when trying to connect to the droplet through the DigitalOcean console. If you face the same problem, I recommend using PuTTY as an alternative. PuTTY is a free SSH client that works well for connecting to your droplet.

![image](https://github.com/user-attachments/assets/04158d34-d68f-4ea8-86a6-b34ab5a57e61)

Once you're connected to the Wazuh Manager droplet, it's time to install Wazuh. But first, make sure to update and upgrade your system:
```
sudo apt update
sudo apt upgrade
```
After your system is updated, you can proceed with the Wazuh installation by running the following commands:
```
curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh
sudo bash wazuh-install.sh -a
```
After the installation is complete, take note of the username and password generated for the Wazuh web interface. You’ll need these credentials to log in later.

To finish setting up Wazuh, open your web browser and enter the public IP address of your Wazuh Manager VM. This will take you to the Wazuh web interface where you can complete the installation process and start configuring your security monitoring environment.

![image](https://github.com/user-attachments/assets/70efea8e-8630-490f-94d5-48a742afb75d)

**TheHive Installation**
TheHive is a scalable, open source and free Security Incident Response Platform, tightly integrated with MISP (Malware Information Sharing Platform), designed to make life easier for SOCs, CSIRTs, CERTs and any information security practitioner dealing with security incidents that need to be investigated and acted upon.

For the TheHive server, follow the same process as you did for the Wazuh Manager droplet. This includes creating and configuring the droplet, assigning the firewall, and updating the operating system.

![image](https://github.com/user-attachments/assets/df7c0f40-ab63-49b7-9109-05531c0aa3fe)

The only difference at this stage is that you'll be installing TheHive instead of Wazuh on this system. TheHive installation instructions:
1. Install dependencies:
   ```
   apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl software-properties-common python3-pip lsb-release
   ```
2. Install Java:
    ```
    wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
    echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
    sudo apt update
    sudo apt install java-common java-11-amazon-corretto-jdk
    echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
    export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
    ```
3. Install Cassandra:
   ```
   wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
   echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
   sudo apt update
   sudo apt install cassandra
   ```
4. Install ElasticSearch
    ```
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
    sudo apt-get install apt-transport-https
    echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
    sudo apt update
    sudo apt install elasticsearch
    ```
    Optionally:
       Create a jvm.options file under /etc/elasticsearch/jvm.options.d and put the following configurations in that file.
   ```
   -Dlog4j2.formatMsgNoLookups=true
   -Xms2g
   -Xmx2g
   ```
5. Install TheHive
    ```
    wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
    echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
    sudo apt-get update
    sudo apt-get install -y thehive
    ```
For more details you can check TheHive installation guide on their official website https://docs.strangebee.com/thehive/installation/step-by-step-installation-guide/

### 2. Thehive configuration
We installed alot of services for our TheHive, now we need to configure it.

#### 2.1 Cassandra Configuration
We'll start by configuring Cassandra, which is used for storing NoSQL data.
1. Edit the Configuration File: Open the Cassandra configuration file located at /etc/cassandra/cassandra.yaml.

![image](https://github.com/user-attachments/assets/1eb042e5-c8a6-47e6-bf40-a940855fbcaa)

2. Find **listen_address** variable and then set the address to the public IP address of your TheHive server

![image](https://github.com/user-attachments/assets/9251032c-944d-42fe-bda7-f70ec5c5fc16)

3. Update the rpc_address: Set the rpc_address to the public IP address of your TheHive server.

![image](https://github.com/user-attachments/assets/6e18520a-dc37-48a9-81f1-7eecbad3541e)

3. Configure the Seed Provider: Locate the seed_provider section and update the seeds parameter with the public IP address of your TheHive server.

![image](https://github.com/user-attachments/assets/dd283e63-b84e-4725-91d8-f841ec056155)

4. Save and Stop the Cassandra Service: After making these changes, save the file and stop the Cassandra service. This will prepare it for a clean start. ```systemctl stop cassandra.service```
5. Cleanup Old Files: Since TheHive was installed from its package, you may need to remove any old or conflicting files that could interfere with the new setup. ```rm -rf /var/lib/cassandra/*```
5. Start the Cassandra Service: Once the cleanup is complete, restart the Cassandra service to apply the changes. ```systemctl start cassandra.service```
6. Verify the Configuration: To ensure everything is working correctly, check the status of the Cassandra service: ```systemctl status cassandra.service```


#### 2.2 ElasticSearch Configuration
Next we will configure ElasticSearch used for querying data.
1. Edit the Configuration File: Open the ElasticSearch configuration file located at /etc/elasticsearch/elasticsearch.yml.

![image](https://github.com/user-attachments/assets/d97b7bf9-dec5-49c4-a3ae-f61de1cf1753)

2. Uncomment **cluster.name** and name it ex. thehive
3. Uncomment **node.name**
4. Uncomment **network.host** and put public ip address of TheHive VM
5. Uncomment **http.port**, by default ElasticSearch uses port 9200
6. ElasticSearch requires either **discovery.seed_host**s or **cluster.initial_master_nodes** to be set to start up properly. We will uncomment cluster.initial_master_nodes and because we work in micro environement we will remove node-2, leaving only node-1 (or whatever name you set for your node).

Save changes and enable elasticsearch
```
systemctl start elasticsearch
systemctl enable elasticsearch
```

![image](https://github.com/user-attachments/assets/7de2f599-5ec0-40c8-9520-4ace3075ae81)

#### 2.3 TheHive configuration

Before we actually start configuring TheHive we need to ensure that thehive user and group have has access to the thehive path, this will allow TheHive to access it's storage directory.
```
ls -la /opt/thp
```

![image](https://github.com/user-attachments/assets/5a1b37ab-a6fd-4a69-a2ea-fa8d481eb5ac)

As you see, root user and group have access to thehive, we want to change this.
```
chown -R thehive:thehive /opt/thp
```
This will change owner of thehive to thehive user and group

![image](https://github.com/user-attachments/assets/dfc090a1-a8d4-43cd-bb1e-516e18102d3a)

Now it's time to access TheHive configuration file under /etc/thehive/application.conf

![image](https://github.com/user-attachments/assets/29299c67-bca1-49bf-a994-84d747d94a48)

1. Change hostname variables to public ip address of TheHive
2. Rename cluster-name to the same from Cassandra
3. Change in application.baseUrl "localhost" to the your VM public IP address

Save changes and enable TheHive
```
systemctl start thehive
systemctl enable thehive
```

Now we can access our Hive web interface by visiting URL of TheHive. In case of issues check:
- If Cassandra service is running
- If ElasticSearch service is running
- If TheHive is running
Use in your browser public ip address of VM with port 9000 to access TheHive interface

![image](https://github.com/user-attachments/assets/9340d764-baec-474b-82a1-1af37939c059)

Default credentials are:
- Login: admin@thehive.local
- Password: secret

### 3. Wazuh Configuration
Wazuh is a free and open source platform used for threat prevention, detection, and response. It is capable of protecting workloads across on-premises, virtualized, containerized, and cloud-based environments. It unifies XDR and SIEM protection for endpoints and cloud workloads.

#### 3.1 Connect Win10 Machine with Wazuh Manager

![image](https://github.com/user-attachments/assets/917e4637-0f8d-4411-8a20-644e0afb11b2)

Access Wazuh Dashboard with credentials you stored earlier. 

![image](https://github.com/user-attachments/assets/cd3bf228-a7b1-42b7-b3b2-829480e00df7)

In Dashboard click **add agen**t and fill up installation process
1. Select Windows Machine
2. For server address put public ip address of Wazuh Manager
3. Assign agent name
4. Copy provided command and execute in on Windows 10 Virtual Machine
5. Start the agent on Win10 VM

![image](https://github.com/user-attachments/assets/19c9d0eb-cdce-4230-bc41-acbc22d38bf9)

![image](https://github.com/user-attachments/assets/089f2d02-f7dc-40b3-b169-8633afe4383b)

Our WazuhAgent and Manager connected successfully, good job!

#### 3.2. Configuring Sysmon Log Ingestion for Wazuh
Now that the Wazuh agent on your Windows 10 machine and the Wazuh Manager are connected, the next step is to configure the agent's telemetry and log ingestion to be sent to the Wazuh Manager.

Access the Configuration File: On your Windows 10 machine, navigate to: 
`This PC -> Local Disk (C:) -> Program Files (x86) -> ossec-agent -> ossec.conf`

The ossec.conf file is the primary configuration file for the Wazuh agent. Open the ossec.conf file with Administrative Privileges by right-clicking on the file and selecting **Run as administrator**. You can begin configuring the agent to send the required telemetry data and logs to the Wazuh Manager for analysis.

![image](https://github.com/user-attachments/assets/2e8d5eca-2541-441b-86be-164285a241ec)

In the Log Analysis section of the ossec.conf file, we can see the general syntax Wazuh uses to define log sources. Now, we'll configure Wazuh to ingest Sysmon logs specifically.
Before making any changes, it's always a good idea to create a backup of the `ossec.conf` file in case something goes wrong.

Next, we need to tell Wazuh where to find the Sysmon logs. To do this:
- Open Event Viewer on the Windows 10 machine.
- Navigate to Applications and Services Logs -> Microsoft -> Windows -> Sysmon.
- Right-click on Operational, and select Properties.
- Look for the Full Name field. The value will be: Microsoft-Windows-Sysmon/Operational. This is the channel we need to configure in Wazuh for Sysmon logs.
- Now that we have the Sysmon log channel, we can configure Wazuh to ingest it.

![image](https://github.com/user-attachments/assets/8b1a6dd8-0668-4f89-bca0-5e2119285896)

Copy the relevant portion of the Log Analysis section in the ossec.conf file.
Update it to target the Sysmon log channel.

For this project, we only want to ingest Sysmon logs and not the default logs like Application, Security, and System logs. Normally, you'd want to keep these for a comprehensive analysis, but for the sake of this project, you can remove or comment out these sections from `ossec.conf`.

Final step is to restart Wazuh. Headover to Services -> Wazuh -> Right Click and then Restart. Remember, it's essential to restart the service every time you modify its configuration to ensure the changes take effect.

![image](https://github.com/user-attachments/assets/73f56e01-7b47-4b32-aa8d-ac2ef007a9c1)

Now we can see Sysmon Events in our Wazuh Dashboard.

#### 4. Mimikatz Test
Mimikatz is a well-known exploit used on Microsoft Windows to extract passwords stored in memory, typically deployed by attackers or red teamers after compromising a system. In this project, we will configure our Wazuh setup to detect Mimikatz activity on the endpoint (our Windows 10 agent) to catch any potential attackers.

![image](https://github.com/user-attachments/assets/23e24ae7-0ea8-462a-8a9b-37c56f5d6321)

By monitoring and analyzing system logs through Wazuh, we can identify behaviors or patterns related to Mimikatz and receive alerts when such activity is detected. 

Mimikatz: https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919

Before downloading Mimikatz for testing, we need to prevent Windows Defender from blocking it. Since Mimikatz is a widely recognized and flagged tool due to its malicious use cases, it will almost certainly be detected and removed by Windows Defender.

To exclude the download path for Mimikatz:
1. Open Windows Security: Go to Start and type Windows Security.
2. Navigate to Virus & Threat Protection: Click on Virus & threat protection from the Windows Security dashboard.
3. Manage Settings: Under Virus & threat protection settings, click on Manage settings.
4. Add an Exclusion:
    - Scroll down to Exclusions and click **Add or remove exclusions**.
    - Select **Add an exclusion**, then choose **Folder**.
    - Select **Downloads** folder

![image](https://github.com/user-attachments/assets/468ddb8d-cda1-4472-99f8-33be21fc6eb1)

Extract Mimikatz in the excluded folder to ensure that Windows won't delete any of the files. Open up the Powershell as  administrator and move to Mimikatz directory

![image](https://github.com/user-attachments/assets/4c77a002-06ed-4b62-8234-d41c3cf6b19d)

Execute mimikatz.exe
`.\mimikatz.exe`

![image](https://github.com/user-attachments/assets/7b50b86a-7ebb-4ca4-8621-58eabba74175)

Head over to the Wazuh Manager web interface and check the logs for any suspicious activity related to Mimikatz. Wazuh may detect unusual behaviors, such as credential dumping or process manipulation.

![image](https://github.com/user-attachments/assets/de7c1ec5-c858-40b6-9b38-21dbe428e1b9)

It may not specifically identify the process as Mimikatz at first glance.

![image](https://github.com/user-attachments/assets/42009caf-b6bf-4168-a702-0ac2d86a0c4c)

To improve detection and explicitly flag Mimikatz execution in the logs, we will need to modify Wazuh's configuration. Wazuh uses rules and decoders to analyze log data, so we will create or modify a custom rule that will explicitly detect Mimikatz.

### 5. Modify Wazuh Configuration
#### 5.1 Log All Events
Go to the Wazuh Manager VM console and first make a backup of configuration file ```cp var/ossec/etc/ossec.conf ~/ossec-backup.conf``` then open `ossec.conf file` there change `<logall>` and `<logall_json>` to **yes**

![image](https://github.com/user-attachments/assets/358335aa-9196-4214-b1df-73e545172dcd)

After this save and then restart Wazuh by ```systemctl restart wazuh-manager.service```. 
This will force Wazuh to put all the logs to the archives, we can find them in `/var/ossec/logs/archives/`

![image](https://github.com/user-attachments/assets/ca2ad18b-6868-4000-96dd-ec77e97f9cec)

In order to make Wazuh ingest these logs we must change configuration in filebeat `nano /etc/filebeat/filebeat.yml` change archives enabled to **True** then restart filebeat service

![image](https://github.com/user-attachments/assets/31795229-5bd1-49db-934a-be64c4e2cb41)

Headover to Wazuh Web Interface, we will create new indexes for our archives. Dashboards Management -> Index patterns

![image](https://github.com/user-attachments/assets/7e2df678-fcaf-437c-a4bc-64cf2959a81c)

![image](https://github.com/user-attachments/assets/4836a385-c452-4780-abab-bdddcc90b72c)

![image](https://github.com/user-attachments/assets/f2207a65-787c-4c1a-8b70-64a5bceaa898)

Now go to Discover and select archives as index to search through logs

![image](https://github.com/user-attachments/assets/4be42dda-3b2e-4574-aa5c-81ecf90ff9b1)

By default, Wazuh only displays logs that trigger predefined rules. However, we’ve configured it to log everything, allowing us to capture and analyze the execution of mimikatz.exe. Now that we can see Mimikatz activity, we can create a custom alert specifically for its execution.

#### 5.2 Custom Wazuh rule creation
When creating a detection rule, it’s more reliable to use data.win.eventdata.originalFileName rather than data.win.eventdata.image
.The reason for this is that attackers often rename files like Mimikatz.exe to Mimikats.exe to evade detection. By focusing on the process name rather than the full command, we can still detect Mimikatz activity even if the executable is renamed.

![image](https://github.com/user-attachments/assets/dd2b7cfb-6d76-4faa-953b-109065ca522d)

Headover to rules, there we will create our custom rule. Click on **manage rule files**

![image](https://github.com/user-attachments/assets/1b1d6862-6c0d-4661-90ef-43c0dff471c4)

We are interested in sysmon events id 1. 

![image](https://github.com/user-attachments/assets/2e4214f0-6d6b-42d9-b025-3612e40bbfd6)

We will copy one of these to build custom rule to detect mimikatz

![image](https://github.com/user-attachments/assets/417f6724-dc8e-4ca8-8f16-364eab4baf40)

Click on custom rules and edit local_rules. Paste copied rule below one already existing.

![image](https://github.com/user-attachments/assets/45a1738c-7ee1-4dd2-a5ad-4c6e4cd02b0b)

- Change id to 100002, custom rules start from 100000
- Change Field name to win.eventdata.originalFileName and then type which is basically regex so it will catch mimikatz.exe
- Set MITRE as T1003 which means Credential Dumping

Let's test our rule against Mimikatz again. I will rename it to `Mimicool` this will prove that we can now detect mimikatz even if it's name is changed by attacker.

![image](https://github.com/user-attachments/assets/81a6eb17-dc91-463f-955d-67bf112b1401)

![image](https://github.com/user-attachments/assets/1e8f2a95-4214-48f9-b784-a8cb78852dc3)

We caught attempt to use mimikatz on our Wazuh agent, congratulations!

### 6. Shuffle Configuration
Final objective which will finalise this project with fully functional lab is to connect Shuffle which will send alerts to TheHive and email to SOC Analyst with response question
What is Shuffle?

![image](https://github.com/user-attachments/assets/82e2e121-9956-45db-b73d-bb9d08b05543)

Shuffle is a no-code automation platform (SOAR) that lets you automate your processes. It is  focused on making automation accessible to everyone, using AI + a simple drag-and-drop interface (with code editing for advanced users). https://shuffler.io/

If you haven't already, head to Shuffle, create a new account, or log in if you already have one. Then navigate to the Workflows section and click on Create New Workflow. This will bring up the workflow editor, where you’ll design the automation. 
- Click on the Triggers section on the left panel and drag the Webhook trigger into your workflow canvas.
- Name the webhook trigger, for example, "Wazuh-Alerts".
- The most crucial part here is to copy the Webhook URL. You'll need this URL later to add it to the Wazuh Manager’s ossec.conf file, enabling Wazuh to send alerts to Shuffle.
- In your workflow, after setting up the webhook trigger, check on Shuffle Tools to see what action follows.
- Check if action is set on "Repeat back to me". Change the call to `$exec`

Now, log into your Wazuh Manager and open the ossec.conf file on the Wazuh Manager. We will add the webhook URL from Shuffle into the configuration to allow Wazuh to send its alerts directly to the Shuffle workflow. Paste integration tag to ossec.conf:
```
<integration>
  <name>shuffle</name>
  <hook_url>http://<YOUR_SHUFFLE_URL>/api/v1/hooks/<HOOK_ID></hook_url>
  <level>3</level>
  <alert_format>json</alert_format>
</integration>
```
We need to make some changes
- Change `http://<YOUR_SHUFFLE_URL>/api/v1/hooks/<HOOK_ID>` to your webhook url from Shuffle
- Change `<level>` to `<alert_id>` and put id number of custom rule in Wazuh. We want to track our custom rule for mimikatz activity from previous steps

![image](https://github.com/user-attachments/assets/e9eb17f6-4e18-4501-bf3a-1a0a6a54c3bc)

**Testing the Wazuh and Shuffle Setup**
Check if the Webhook trigger is active. If not, click the "Start" button to activate it. Then go to the Windows 10 Agent and execute Mimikatz.exe. After Mimikatz has been executed, switch back to Shuffle. Click on the icon of the running person, labeled "Show Executions", in your workflow dashboard. Check if Shuffle has caught and processed the Mimikatz execution alert sent from Wazuh.

![image](https://github.com/user-attachments/assets/bd6ec38c-baa2-4934-adda-39d0c075441f)

If everything is set up correctly, you should see the alert for the Mimikatz execution in Shuffle, confirming that the webhook integration works.

### 7. Building Workflow
Now that everything is set up, let’s outline how we can expand the Shuffle workflow for our project to fully automate the alerting and response process
1. Mimikatz Alert Sent to Shuffle:
    - Wazuh detects the Mimikatz execution and triggers an alert.
    - The alert is sent to Shuffle using the Webhook we previously configured.
2. Shuffle Receives Mimikatz Alert and Extracts SHA256 Hash:
    - Once Shuffle receives the alert, it extracts the SHA256 hash of the Mimikatz executable from the Wazuh alert data.
    - The workflow can parse the incoming JSON alert and pull out relevant fields, including the file hash.
3. Shuffle Checks Reputation Score with VirusTotal:
    - Using the extracted SHA256 hash, Shuffle sends a request to VirusTotal API to check the reputation of the file.
    - VirusTotal responds with a reputation score, detailing whether the file is known to be malicious, along with other metadata.
4. Send Details to TheHive to Create Alert:
    - Based on the VirusTotal response and the original Wazuh alert, Shuffle sends all relevant details (such as the file hash, reputation score, and alert metadata) to TheHive.
    - TheHive automatically creates a new alert or case to begin the investigation process.
5. Send Email to SOC Analyst:
    - Shuffle sends an email notification to a SOC Analyst with the alert details, including the VirusTotal results and a link to TheHive case.
    - The email serves as a trigger for the analyst to start investigating the alert.
  
#### 7.1 Extracting Hash Value

![image](https://github.com/user-attachments/assets/f42a5295-3a4a-434c-abbf-0a9810c7f4cc)

To extract hash value with format "SHA=XXX", change Action from "Repeat back to me" to "Regex Capture Group". Click on + button and select "execution argument" then find hash value
For Regex itself use `SHA1=([A-Fa-f0-9]{40})`. Now to test this, just save workflow and rerun it by clicking "Rerun Workflow" in "Show Executions"

![image](https://github.com/user-attachments/assets/6f644b6a-11d6-460d-9d53-2c4e3decd056)

As you see, we just copied hash value. When i copy it to Virustotal it successfuly finds Mimikatz.exe. This means our regex capture is correct.

![image](https://github.com/user-attachments/assets/85534077-6cf5-41ca-9919-a17be3de2b0b)

To finalise this step rename Shuffle Tools icon to "SHA_Regex"

#### 7.2 Check Reputation Score with VirusTotal
Next step is to send this hash to VirusTotal API to automatically extract reputation score of this hash value. To utilise their API we must create account in VirusTotal. Then copy our API Key and headover to the Shuffle. Add VirusTotal to workflow by clicking "Apps", searching for VirusTotal, activating it and then draging it to the dashboard


![image](https://github.com/user-attachments/assets/639eacde-502d-4422-9de7-0cf6faf5c6dd)

- Insert your API Key or authenticate with your VirusTotal.
- Select Hash Report action
- For ID select our SHA Regex list

![image](https://github.com/user-attachments/assets/56eb3833-e1e2-402e-910e-bcb6d3ed6960)

Again to test this, just save workflow and rerun it by clicking "Rerun Workflow" in "Show Executions"

![image](https://github.com/user-attachments/assets/549b295f-6ca6-4b0b-9003-4247c1d729ea)
![image](https://github.com/user-attachments/assets/501806a0-85be-45c3-9c8b-4e11bf2ca695)

#### 7.3 Send details to TheHive
Similarly to VirusTotal, add TheHive app to the Workflow and connect VirusTotal to TheHive, to authenticate us, we will get API key from the TheHive Dashboard. Log in to TheHive web interface and copy this API key. But first let's create new organization and user. 

![image](https://github.com/user-attachments/assets/a9a2cb6e-4473-46a8-a722-1c8a71c3f95e)

Click on + button in TheHive dashboard to create new organization.

![image](https://github.com/user-attachments/assets/8ca2efa7-bebf-4475-a8a5-9a36e885c525)

Then click on the organization to add users. We will create 2 users
- Normal User, with analyst profile
- Service User with analyst profile

![image](https://github.com/user-attachments/assets/e78ffb8f-65d1-41e6-8cc2-1e448749161e)

**Note: In this demo we are using default profiles, in real environement we should create new profiles with principle of least privileges, that allows to perform all neccessary tasks with minimal permissions.**

For our normal user account we want to create password, click on preview and scroll down to create new strong password.
For service user we want to create API key.

Now log in using normal user account. You will see different dashboard with cases and alerts.

![image](https://github.com/user-attachments/assets/676f42cc-0da4-4fe7-9abc-8ae4b82b4ed3)

Return to the Shuffle and authenticate TheHive app using API key. For url put url of your TheHive with port number

![image](https://github.com/user-attachments/assets/7a2b1b7d-edbe-4521-9656-9436dfdbf349)

Now under actions we want to create alert.

Fill up the body as you wish, this contains information for our alert. I did this like this:

![image](https://github.com/user-attachments/assets/a19fa1c8-f743-485c-8cc9-0bbf1f642a75)


For TheHive to receive action about creating alert we must edit it's firewall to allow coming traffic through port 9000. To inbound Rules add rule:

![image](https://github.com/user-attachments/assets/fa7a0735-d3e9-48a8-81d3-5dc49b479bf2)

Now we can rerun the workflow!

![image](https://github.com/user-attachments/assets/59a9409c-a03f-4872-9b96-2af99e859d3b)
![image](https://github.com/user-attachments/assets/eaa98ea3-c295-4dc6-8c3b-478d74a11aa3)

Success! Shuffle automatically created alert in TheHive for further investigation

#### 7.4 Send email to the Analyst
Next step is to send email to the security analyst containing relevant information.

Drag in and drop email application and connect VirusTotal to Email
- You can set as recipents your active email, i will personally use SquareX functionality to create temporary, disposable email address.
- Fill subject and body with valuable data for the analyst.
Rerun the workflow and done! We received email that alerts us about potential attacker.

![image](https://github.com/user-attachments/assets/f28721d6-6e6c-4b1a-a129-a151dae71459)

This concludes this part of the project. I hope this walkthrough was helpful to you. Good luck, and happy building!

### 8. Stopping SSH Bruteforce Attempts
This is a bonus section of the project, where we will review and consolidate our knowledge by building a workflow around a new issue on a different agent machine. This will allow us to practice the skills we've learned so far. The goal is to create an automated response to SSH brute force attempts. When such an attempt is detected, an email will be sent to the analyst with details of the attack, asking whether the IP address should be blocked. If the analyst approves, the IP will be automatically blocked.

For this task, we will also need an attacker's machine. I'll use a Kali Linux VM installed on my PC. In the future, we'll configure the Ubuntu Agent Machine to allow all inbound traffic, enabling us to test our SOAR (Security Orchestration, Automation, and Response) setup against real SSH brute force attempts.

Before we proceed, we will extract the Wazuh API user login and password. These credentials can be found in the wazuh-install-files.tar archive. To extract it, use the following command:
```
tar -axf wazuh-install-files.tar
```
Once extracted, open the wazuh-passwords.txt file and locate the API user credentials. Make sure to save them for later use in the project.

#### 8.1 Build and Configure Ubuntu Machine
Similary like before, i will set up Ubuntu VM on Digital Ocean, i will use minimal possible specifications for this machine. For now set it's Firewall to allow traffic from your public IP address and public IP address of Wazuh Manager. Remember to update and upgrade packages of this machine.

![image](https://github.com/user-attachments/assets/f9cc345a-84e8-4fcc-ab1e-a80d98a9614d)

##### **Install Wazuh Agent on Ubuntu**
First, we will allow traffic from the Ubuntu Agent to reach the Wazuh Manager. Currently, traffic is only allowed from our public IP address. This isn't an issue if you're using an on-premise VM, but in a cloud environment, we need to ensure the Ubuntu VM can communicate with the Wazuh Manager.

![image](https://github.com/user-attachments/assets/aefecf46-ce8d-4b22-864a-306d6ae3350e)

To install the Wazuh Agent on Ubuntu, the process is similar to installing it on Windows. Start by accessing the Wazuh Manager Web Interface and selecting the "Add Agents" option. Follow the instructions provided by Wazuh, then copy the generated command to the Ubuntu Agent. After completing this process, the Wazuh Agent will be fully installed on the Ubuntu system.

![image](https://github.com/user-attachments/assets/0c1f3a89-d8c7-48af-b602-b7bf9d9b92f5)

Now we can perform little test, hop to your attacker machine and perform hydra dictionary attack on the Ubuntu Agent then we will check how Wazuh interprets this attack attempt.

![image](https://github.com/user-attachments/assets/d2f35649-2fe7-4e18-b910-ada8bcdd89d4)

![image](https://github.com/user-attachments/assets/e08f0a16-fc05-45fc-9a20-7059b0ef4216)

To detect SSH brute force attempts in Wazuh while minimizing noise, it’s crucial to focus on specific rule IDs that are highly indicative of brute force attacks and to avoid redundancy by filtering out rules that may trigger for the same event. We will select:
- Rule ID 5551 - PAM: Multiple failed logins in a small period of time.
- Rule ID 5712 - sshd: Attempt to login using a non-existent user

#### 8.2 Push SSH Brute Force attempt alerts to Shuffle
Similary like before we add Webhook to the workflow with

![image](https://github.com/user-attachments/assets/60ff02be-6d27-47a4-9aa3-515f61e3f75c)

This time integration tag in ossec.conf of Wazuh Manager will catch 2 rules we choose for SSH Brute Force Attempts:
```
<integration>
  <name>shuffle</name>
  <hook_url>http://<YOUR_SHUFFLE_URL</hook_url>
  <alert_id>5551, 5712</alert_id>
  <alert_format>json</alert_format>
</integration>
```

Now execute Hydra again and then run this Workflow in Shuffle. You will catch event that activated this rule.

![image](https://github.com/user-attachments/assets/200cdef7-3535-4b4a-a92f-5ad9fa5bea8b)

#### 8.3 Perform IP Enrichment with VirusTotal
In Shuffle, configure the webhook to connect with VirusTotal. Choose the action “Get an IP address report” for VirusTotal. If necessary, authenticate with the VirusTotal API. In the IP section of the VirusTotal action configuration, set the source IP to be the alert IP received from execution arguments. This ensures that the correct IP address is checked.

Execute the workflow again to verify that the VirusTotal reputation check is functioning correctly.

![image](https://github.com/user-attachments/assets/ee8a7386-729e-454e-9b83-0204696bb1dd)

Since we are currently using a public IP that isn't malicious, you may not see significant results. When you catch real SSH brute force attempts, you may identify malicious IP addresses through VirusTotal.

#### 8.4 Set up Automatic Response
To begin, Shuffle needs access to the Wazuh API, which will enable various new functions. We can achieve this by sending a curl request to the Wazuh API using the Wazuh API user credentials. This request will extract a JSON token from Wazuh, allowing Shuffle to make subsequent requests to the Wazuh Manager.

Drag the HTTP app into your workflow in Shuffle. Set the action to curl then Insert the following command in the statement field:
```
curl -u <WAZUH_API_USER>:<WAZUH_API_PASSWORD> -k -X GET "https://localhost:55000/security/user/authenticate?raw=true"
```
Replace `<WAZUH_API_USER>` and `<WAZUH_API_PASSWORD>` with the credentials extracted from the wazuh-passwords.txt file
Command will return a JSON token that you can use for authentication to the Wazuh Manager API.

To ensure that Shuffle can communicate with the Wazuh Manager, you need to allow traffic on port 55000. This port is used by the Wazuh Manager for API requests. Configure Firewall like we set up earlier.

![image](https://github.com/user-attachments/assets/e1428b3c-38ba-46d9-ac53-59de1f094040)

After running workflow you should retrieve JSON Token.

Now let's begin with building up Automatic Response.
Drag in the Wazuh Application:
- Set the action to **Run a command.**
- Use the API key retrieved from the Get_API HTTP app.
- For the Agent list, use the argument that extracts the agent_id from the alert.
- Set Wait for completion to `True`.

**Configure Active Response in Wazuh Manager:**
We need to go to Wazuh Manager and edit the `ossec.conf` file. In the Active response section, you'll find the list of commands that can be invoked. We are interested in `firewall-drop`.  Scroll down to the <active-response> section under commands and modify it like this:
```
  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rule_id>5551,5712</rule_id>
    <timeout>no</timeout>
  </active-response>
```
The `firewall-drop` command will be invoked on the local machine where Wazuh detects SSH brute force attempts, specifically matching rule IDs 5551 and 5712. The `firewall-drop` action will modify the iptables of the agent to block the source IP address responsible for the SSH brute force attempts. **Remember to restart wazuh manager!**

Now to use this command via API we must append to it timeout value, in case of "no" value is 0 which means command is: `firewall-drop0`
You can check that in `/var/ossec/bin/agent-control.

![image](https://github.com/user-attachments/assets/8385cdf4-521b-4ba5-88cb-476a69a33bc1)

![image](https://github.com/user-attachments/assets/bbcbfede-037d-4a9c-bbb4-c300f282e9d3)

Now return to Shuffle and under body write:

![image](https://github.com/user-attachments/assets/decefb92-f845-4a1d-a5b0-be92bf23fed2)

Now, we are using the source IP 8.8.8.8 to test the workflow, which would block Google’s IP. In practice, we will configure the execution argument to automatically extract the source IP of the malicious request, ensuring the right IP is blocked based on the alert.

After running the workflow, you should see the updated iptables configuration on your Ubuntu agent by running: `iptables --list`:

![image](https://github.com/user-attachments/assets/8fde00cb-867b-4639-999d-aa5b8a484fb0)

#### 8.4 Send details via Email to the User 
We also want to notify the analyst via email, asking them whether they want to block the IP address. We will enrich the email with data from VirusTotal to provide the analyst with better context for making a decision.

In your workflow, drag the User Input app.
Set the input method to "email" and use the email address that was set earlier in your squareX setup.
In the information field of the email, you can provide relevant data. Write the following message:
```
Would you like to block this source IP: $exec.all_fields.data.srcip
```
This will display the source IP for the analyst to review, you can add more information if you wish.

Once the workflow is set, rerun it and ensure that the email is sent as expected.
![image](https://github.com/user-attachments/assets/f75bc9b8-8b8a-4619-8efd-a3a1ab710205)


![image](https://github.com/user-attachments/assets/3365a55f-d093-4f93-a90b-4125823bc678)

I received email asking for my input. Since I am currently using my public IP from a Kali VM for testing, it will display this IP in the email. You don’t want to block your own IP in this case. 

#### 8.6 Test against real threats
To test with real data, allow all inbound traffic to your Ubuntu agent. This will enable you to capture actual SSH brute force attempts, triggering the alert and workflow for blocking malicious IP addresses.

Create special firewall for ubuntu agent which will allow all inbound traffic. This will expose machine to the internet.

![image](https://github.com/user-attachments/assets/9718efcb-b720-4b83-8716-df070860c1b3)

Now let's test our workflow against this ip address!

![image](https://github.com/user-attachments/assets/22650d71-0a68-437e-b340-958457a7d510)
![image](https://github.com/user-attachments/assets/beeddace-59cd-4378-9ade-a1b616369c9f)
![image](https://github.com/user-attachments/assets/16b8426a-6612-4986-89a2-07c846507b7c)
![image](https://github.com/user-attachments/assets/2fa8ee54-a331-425a-8744-11caa475c26c)
![image](https://github.com/user-attachments/assets/44b0f883-dbc3-4cb5-bfa7-c2a477328071)

We have successfully detected a real SSH brute force attempt using our workflow. An email was sent to the analyst with a question on whether to block the suspicious IP address. After responding with "yes," Shuffle sent a request to Wazuh, which in turn blocked the IP on the Ubuntu agent

Congratulations! 🎉
We have now implemented a fully functional automatic response to SSH brute force attacks. This marks the completion of the setup for automatically detecting and mitigating such threats!

## Sources:
- Mydfir Channel - [https://www.youtube.com/@MyDFIR](https://www.youtube.com/watch?v=Lb_ukgtYK_U&list=PLEd_qaF8wpnXgdngqfsQtYYGM-IdtuxmC)
- TheHive Documentation https://docs.strangebee.com/
- Wazuh Documentation https://documentation.wazuh.com/current/index.html
- Shuffle Documentation https://shuffler.io/docs/about
