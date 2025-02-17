# Active Directory & Security Lab Documentation

This document details my journey in building a virtual security lab inspired by the myDFIR YouTube course ([Watch the Course](https://www.youtube.com/watch?v=5OessbOgyEo)). The lab simulates an enterprise network environment using Oracle VM VirtualBox, where I deployed multiple operating systems to practice cybersecurity techniques from network configuration and Active Directory management to endpoint monitoring and simulated attacks.

---

## Overview

**Lab Environment Components:**

- **Windows 10 Client:** A typical business workstation.
- **Kali Linux Attacker:** A penetration testing platform for launching simulated attacks.
- **Windows Server 2022:** Configured as an Active Directory Domain Controller (AD DC) to provide centralized authentication and directory services.
- **Ubuntu Server:** Serving as the Splunk log analysis server and overall security monitoring station.

**Key Security Solutions:**

- **Splunk Enterprise & Universal Forwarder:** For aggregating, analyzing, and visualizing logs.
- **Sysmon:** For detailed tracking of endpoint activities on Windows systems.
- **Crowbar:** To mimic brute force attacks.
- **Atomic Red Team (ART):** For running diverse security tests and validations.
- **PowerShell Scripting:** Used extensively to automate configuration and management tasks throughout the lab.

**Estimated Setup Duration:** Approximately 3-4 hours

---

## Objective

The primary goal of this lab is to provide a practical, hands-on experience in configuring a virtualized environment for cybersecurity exploration. By deploying and managing VMs running Windows 10, Kali Linux, Windows Server, and Ubuntu Server, I learned how to:

- Configure network settings (static IPs, NAT networks, DNS troubleshooting).
- Install and fine-tune security tools like Splunk and Sysmon.
- Simulate security attacks using tools such as Crowbar and ART.
- Manage an Active Directory domain, including joining clients and enabling remote access.
- Automate routine tasks with PowerShell to streamline operations.

This lab serves as a controlled setting where theoretical cybersecurity concepts are put into practice, reinforcing both offensive and defensive security skills.

---

## Skills Acquired

- **Virtualization:** Creating and managing virtual machines in Oracle VM VirtualBox.
- **Networking:** Setting up NAT networks, static IP addressing, and DNS troubleshooting.
- **Software Deployment:** Installing Splunk Enterprise, Universal Forwarder, and Sysmon.
- **Security Testing:** Simulating brute force attacks with Crowbar and validating defenses with Atomic Red Team.
- **Active Directory Management:** Configuring an AD domain, adding computers, and setting up user accounts.
- **Remote Desktop & Endpoint Monitoring:** Enabling RDP on Windows and monitoring activity with Sysmon.
- **Scripting & Automation:** Utilizing PowerShell commands (e.g., Invoke-WebRequest, Set-ExecutionPolicy) to automate tasks.

---

## Tools and Technologies

- **Oracle VM VirtualBox Manager:** Virtual machine host and management.
- **Splunk Enterprise & Universal Forwarder:** For log analysis, data aggregation, and real-time monitoring.
- **Sysmon:** Advanced endpoint monitoring for Windows.
- **Crowbar:** Tool for brute force password testing.
- **Atomic Red Team (ART):** Framework for executing security tests.
- **PowerShell:** Command-line scripting for automation.
- **Microsoft Windows Event Logs:** Sources for security event monitoring in Splunk.
- **Windows Server 2022:** Platform for Active Directory Domain Services.
- **Ubuntu Server:** Deployed as the Splunk server.
- **Microsoft Windows 10:** Target machine for user operations.
- **Kali Linux:** Platform for offensive security testing.

---

## Detailed Setup and Configuration


![Active Directory Project](https://github.com/user-attachments/assets/4ad274de-c6b0-4c03-9e79-339027cf1159)


### Part 1: Virtual Machine Installation

1. **Install VirtualBox:**
   - Download and install Oracle VM VirtualBox from [virtualbox.org](https://www.virtualbox.org/).

2. **Deploy Windows 10:**
   - Obtain the Windows 10 ISO from the [Microsoft download page](https://www.microsoft.com/en-ca/software-download/windows10).
   - Create a new VM in VirtualBox (e.g., named "target-PC") using the Windows 10 ISO. Allocate 4096 MB of RAM, 1 CPU, and a 50 GB virtual disk.
   - Boot the VM and follow the custom installation process.

3. **Deploy Kali Linux:**
   - Download the VM version of Kali Linux from [kali.org](https://www.kali.org/).
   - Use 7-Zip (available from [7-zip.org](https://www.7-zip.org/)) to extract the image if necessary.
   - Import the Kali Linux VM into VirtualBox and start it.

4. **Deploy Windows Server 2022:**
   - Download the Windows Server 2022 ISO from [Microsoft’s Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022).
   - Create a new VM in VirtualBox with the Windows Server ISO. Allocate 4096 MB of RAM, 1 CPU, and a 50 GB virtual disk.
   - Follow the installation prompts, choose the “Desktop Experience” version, and set an administrator password.

5. **Deploy Ubuntu Server:**
   - Download Ubuntu Server 22.04.4 LTS from [ubuntu.com/server](https://ubuntu.com/server).
   - Create a new VM in VirtualBox with 8192 MB of RAM, 2 CPUs, and a 100 GB virtual disk.
   - Boot the VM, select “Try or Install Ubuntu Server,” complete the installation, and then update the system:
     ```bash
     sudo apt-get update && sudo apt-get upgrade -y
     ```

*After completing these steps, you should have four VMs running: Windows 10, Kali Linux, Windows Server, and Ubuntu Server.*

---

### Part 2: Network Configuration and Splunk Setup

1. **Configure Network Communications:**
   - In VirtualBox, navigate to **Tools > Network > NAT Networks** and create a new NAT network (e.g., with an IPv4 prefix `192.168.10.0/24`).
   - Adjust each VM’s network settings to connect to this NAT network.

2. **Assign a Static IP to Ubuntu (Splunk Server):**
   - Log in to Ubuntu and open the netplan configuration file:
     ```bash
     sudo nano /etc/netplan/00-installer-config.yaml
     ```
   - Update the file as follows:
     ```yaml
     network:
       ethernet:
         enp0s3:
           dhcp4: no
           addresses: [192.168.10.10/24]
           nameservers:
             addresses: [8.8.8.8]
           routes:
             - to: default
               via: 192.168.10.1
       version: 2
     ```
   - Apply the configuration:
     ```bash
     sudo netplan apply
     ```
   - Verify with:
     ```bash
     ip a
     ping google.com
     ```

3. **Install and Configure Splunk Enterprise:**
   - Download the Splunk Enterprise trial (.deb package) from [splunk.com](https://www.splunk.com/).
   - Install VirtualBox guest additions:
     ```bash
     sudo apt-get install virtualbox-guest-additions-iso
     ```
   - Create and mount a shared folder in Ubuntu:
     ```bash
     mkdir share
     sudo mount -t vboxsf -o uid=1000,gid=1000 <shared_folder_name> share/
     ls -la share/
     ```
   - Install Splunk:
     ```bash
     sudo dpkg -i splunk-<version>.deb
     ```
   - Start Splunk as the Splunk user:
     ```bash
     sudo -u splunk bash
     cd /opt/splunk/bin
     ./splunk start
     ```
   - Enable Splunk to start on boot:
     ```bash
     sudo ./splunk enable boot-start -user splunk
     ```

4. **Configure the Windows 10 Client:**
   - Rename the PC (e.g., “target-PC”) and restart.
   - Set a static IP by modifying the network adapter settings:
     - IP Address: `192.168.10.100`
     - Subnet Mask: `255.255.255.0`
     - Default Gateway: `192.168.10.1`
     - DNS Server: `8.8.8.8`
   - Verify the configuration using `ipconfig`.
   - Download and install the Splunk Universal Forwarder from [splunk.com](https://www.splunk.com/), configuring it to forward logs to `192.168.10.10:9997`.

5. **Deploy Sysmon on Windows 10:**
   - Download Sysmon from [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
   - Obtain a configuration file (sysmonconfig.xml) from [sysmon-modular](https://github.com/olafhartong/sysmon-modular).
   - Open PowerShell as Administrator and run:
     ```powershell
     .\Sysmon64.exe -i sysmonconfig.xml
     ```
   - Edit `inputs.conf` in `C:\Program Files\SplunkUniversalForwarder\etc\system\local\` to capture Sysmon events:
     ```
     [WinEventLog://Application]
     index = endpoint
     disabled = false

     [WinEventLog://Security]
     index = endpoint
     disabled = false

     [WinEventLog://System]
     index = endpoint
     disabled = false

     [WinEventLog://Microsoft-Windows-Sysmon/Operational]
     index = endpoint
     disabled = false
     renderXml = true
     source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
     ```
   - Restart the Splunk Universal Forwarder service.

6. **Set Up the Windows Server:**
   - Rename the server (e.g., “ADDC01”) and configure a static IP:
     - IP Address: `192.168.10.7`
     - Subnet Mask: `255.255.255.0`
     - Default Gateway: `192.168.10.1`
     - DNS Server: `8.8.8.8`
   - Install Splunk Universal Forwarder and Sysmon on the server using the same procedure as on Windows 10.
   - Verify that the Splunk web interface at `http://192.168.10.10:8000` displays log data from both the client and the server under the “endpoint” index.

---

### Part 3: Active Directory Configuration

1. **Configure Active Directory on Windows Server:**
   - Open **Server Manager** on Windows Server and choose **Add Roles and Features**.
   - Install the **Active Directory Domain Services (AD DS)** role.
   - When prompted, click the notification flag and select **Promote this server to a domain controller**.
   - Choose to create a new forest and enter the domain name `mydfir.local`.
   - Complete the wizard by accepting the default settings and setting a DSRM password; the server will then restart.
   - After reboot, log in as `MYDFIR\Administrator`.

2. **Create Organizational Units and Users:**
   - Open **Active Directory Users and Computers** from the Tools menu.
   - Right-click on `mydfir.local` and create a new Organizational Unit named **IT**.
   - Within the **IT** OU, create a new user (e.g., Jenny Smith with the username `jsmith`).
   - Optionally, create additional OUs (e.g., **HR**) and users for testing purposes.

3. **Join Windows 10 to the Domain:**
   - On the Windows 10 client, open **Advanced System Properties** > **Computer Name** and click **Change**.
   - Under **Member of**, select **Domain** and enter `MYDFIR.LOCAL`.
   - Adjust the network adapter’s DNS settings to point to `192.168.10.7` (the IP address of ADDC01).
   - Verify the settings with `ipconfig /all`, then join the domain.
   - Reboot the machine and log in using the domain credentials.

---

### Part 4: Simulated Attack with Kali Linux

1. **Configure Kali Linux Networking:**
   - Boot Kali Linux and open the network manager.
   - Set a manual IP: `192.168.10.250`, Netmask: `/24`, Gateway: `192.168.10.1`, and DNS: `8.8.8.8`.
   - Confirm the settings with:
     ```bash
     ip a
     ping google.com
     ping 192.168.10.10
     ```
   - Update the system:
     ```bash
     sudo apt-get update && sudo apt-get upgrade -y
     ```

2. **Prepare for the Brute Force Attack:**
   - Create a directory for the attack project:
     ```bash
     mkdir ad-project && cd ad-project
     ```
   - Install Crowbar:
     ```bash
     sudo apt-get install -y crowbar
     ```
   - Navigate to the wordlists directory, unzip “rockyou.txt.gz”, and copy it:
     ```bash
     cd /usr/share/wordlists
     sudo gunzip rockyou.txt.gz
     cp rockyou.txt ~/Desktop/ad-project
     cd ~/Desktop/ad-project
     head -n 20 rockyou.txt > passwords.txt
     ```

3. **Enable Remote Desktop on Windows 10:**
   - On the Windows 10 client, go to **This PC > Properties > Advanced system settings > Remote**.
   - Enable **Allow remote connections to this computer** and add the necessary domain user(s) (e.g., `jsmith`).

4. **Execute the Brute Force Attack:**
   - On Kali Linux, run the following Crowbar command to target the RDP service:
     ```bash
     crowbar -b rdp -u tsmith -C passwords.txt -s 192.168.10.100/32
     ```
   - If a valid password is found, Crowbar will output the credentials.

5. **Analyze the Attack with Splunk:**
   - Log in to the Splunk web interface at `http://192.168.10.10:8000` and search the “endpoint” index.
   - Look for repeated instances of Event Code **4625** (failed logon attempts) and **4624** (successful logons); a spike in 4625 events indicates brute force attempts.
   - Drill down into event details to identify the attack source.

![Screenshot for brute force attack log(failed attempts)](https://github.com/user-attachments/assets/82e204c9-3f3a-44da-bcf4-622e0174ed76)

6. **Run Tests with Atomic Red Team (ART):**
   - On the target machine, open PowerShell as Administrator.
   - Set the execution policy:
     ```powershell
     Set-ExecutionPolicy Bypass -Scope CurrentUser
     ```
   - Install ART by running:
     ```powershell
     IEX (Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
     Install-AtomicRedTeam -getAtomics
     ```
   - Navigate to the ART directory and execute a test (e.g., for persistence using a local account creation test):
     ```powershell
     Invoke-AtomicTest T1136.001
     ```
   - Verify in Splunk that unauthorized local account creation events are logged, which helps in identifying potential security gaps.

---

## Final Summary

At the conclusion of this project, the lab environment comprises four interconnected virtual machines:

- **Windows 10 Client (target-PC)**
- **Kali Linux Attacker**
- **Windows Server (ADDC01)**
- **Ubuntu Server (Splunk)**

This setup allows for comprehensive security monitoring and testing. I can view detailed logs in Splunk, simulate brute force attacks and other methods with Crowbar and ART, and validate the defensive measures in an Active Directory environment. This hands-on lab has greatly enhanced my understanding of virtualization, network security, endpoint monitoring, and automated system management using PowerShell.

---

## Acknowledgments

I extend my sincere thanks to myDFIR on YouTube for the outstanding course that inspired this project. Their comprehensive tutorials have been instrumental in transforming cybersecurity theory into practical, actionable skills. For further learning and advanced techniques, I highly recommend watching the myDFIR course [here](https://www.youtube.com/watch?v=5OessbOgyEo).
