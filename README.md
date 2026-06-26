# SOC-Automation-Lab-Project
This SOC automation lab simulates how a security team detects, manages, and responds to alerts. Wazuh is used as the SIEM, TheHive is used for case management, and Shuffle is used as the SOAR to automate the workflow. The lab demonstrates endpoint telemetry, alert generation, case creation, analyst notification, and basic incident response automation.

# Lab Objectives
1. Simulate a SOC workflow from detection to investigation and response
2. Configure Wazuh as the main SIEM/XDR platform (using Azure)
3. Configure TheHive as the case management platform (using Azure)
4. Configure Shuffle as the SOAR
5. USe Mimikatz in a Windows VM to generate malicious telemetry  
6. Use logs from Sysmon

# Lab Roadmap
Basic Flow:
```mermaid
flowchart TD
    A["Windows Endpoint"] --> B["Sysmon / Wazuh Agent"]
    B --> C["Wazuh SIEM/XDR"]
    C --> D["Shuffle SOAR"]
    D --> E["TheHive Case Management"]
    E --> F["Analyst Notification / Response Action"]
```

Activity on the Windows endpoint is captured by Sysmon and the Wazuh agent, which send logs to the Wazuh SIEM/XDR for analysis and alert generation. Alerts are then passed to Shuffle, which automates workflows such as creating cases in TheHive. Finally, TheHive organizes the incident for investigation and can trigger notifications or response actions by the analyst.

## Lab Sections
This lab is divided into 4 sections, namely:
### Section 1: Platforms and Endpoint Preparation

Azure virtual machines are created for Wazuh and TheHive, SSH access was configured to configure them. Shuffle is also set up as the SOAR.

### Section 2: Server and Endpoint Configuration

Wazuh, TheHive, and the Windows endpoint are configured so they can communicate with each other.

### Section 3: Telemetry and Alert Generation

This will involve generating endpoint telemetry utilizing Mimikatz and Sysmon and creating alerts in Wazuh.

### Section 4: SOAR Integration and Automation

Integrating Wazuh, TheHive, and Shuffle to automate alert handling, case creation, analyst notification, and possible response actions.

# Section 1: Platforms and Endpoint Preparation
## 1.1. Setup Windows 11 Virtual Machine
A Windows 11 Pro Virtual Machine has been setup in VirtualBox.

![](/screenshots/vm.png)

Sysmon has also been setup and configured in this VM.
![](/screenshots/sysmon_installed.png)

## 1.2 Setup of Wazuh and TheHive in Microsoft Azure  
### 1.2.1 Setup a Resource Group
For this lab Wazuh and TheHive are hosted in Microsoft Azure for convenience and to free up resources in the local machine, as it is already running the Windows VM locally. 

A resource group named "soc-lab-rg" is set up. All our resources including the Wazuh and TheHive servers will be stored here, as well as the virtual networks.
![](/screenshots/azure_rg.png)

### 1.2.1 Wazuh Server Deployment
For the server that will host Wazuh, an Ubuntu 24.04 LTS VM was deployed. Named wazuh-server and was deployed to have 4 vCPUs and 8 GBs of RAM. This server will host our SIEM/XDR. Initially, SSH is enabled within the network settings of this VM to allow access to the VM using our local machine.

We can take note of the assigned public IP address for this server which is:
    
    Wazuh Server Public IP: 20.205.120.231

![](/screenshots/azure_wazuh.png)


### 1.2.2 TheHive Server Deployment
For the TheHive server, a slightly more powerful Ubuntu 24.04 LTS VM was with 16 GB of RAM was used since TheHive requiers heavier backend components and is used for investigation and case tracking. This VM was named thehive. Like the Wazuh server, SSH is also enabled initially for this VM.

We can also take note of the assigned public IP address for this server which is:

    TheHive Server Public IP: 20.89.254.38

![alt text](/screenshots/azure_thehive-1.png)

## 1.3 SSH Access to Azure-based VMs and Update Ubuntu.

### 1.3.1. Acesss Azure VMs using SSH
To access our Azure VMs from our local machine, we can use secure shell or SSH. An SSH key-based authentication was set up to ensure a secure connection.

SSH key-based authentication uses a pair of cryptographic keys: a private key stored securely on the local machine and a public key placed on the server. When connecting, the server verifies the private key without transmitting it over the network. This approach is more secure than password-based authentication.

To SSH onto our Azure VMs, we can use the command:

    ssh -i <PRIVATE_KEY_FILE>.pem <USERNAME>@<SERVER_PUBLIC_IP>

We used Windows Powershell to enter this command to access both servers. The default username for these is "azureuser". It's also important that when performing this command, we must be in the same directory as the .pem or SSH key. As shown in the screenshots below.


SSH to the Wazuh Server:
![alt text](/screenshots/wazuh_ssh.png)

SSH to TheHive Server:
![alt text](/screenshots/thehive_ssh-1.png)

### 1.3.2 Ubuntu Updates
Now that we have SSH connection to both our VMs, we can go ahead and update our Ubuntu package repositories. 

We can use the command:

    sudo apt update && sudo apt upgrade -y

 - sudo: Runs the command with administrative (root) privileges
 - apt update: Refreshes the package list from repositories
 - &&: Ensures the next command runs only if the previous one succeeds
 - apt upgrade: Installs available updates for installed packages
 - -y: Automatically confirms prompts to proceed with the upgrade

Wazuh Server Update:
![alt text](/screenshots/wazuh_ubuntu_update-1.png)

TheHive Server Update:
![alt text](/screenshots/thehive_ubuntu_update-1.png)


With this SSH connection and Ubuntu updates, we now have an established connection from our local machine to our updated VMs that we configured earlier in Azure. We can now install our platforms in these VMs and configure them.

## 1.4 Wazuh Installation
Wazuh was installed on the Azure Ubuntu server using the Wazuh installation assistant.

The main command used is:

    curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

The screenshot below shows that Wazuh has already been installed. This is because I have already installed Wazuh previously, but no harm in still entering the command.

![alt text](wazuh_installing.png)

Wazuh will be used as the main SIEM/XDR component of the lab. It will collect logs, analyze events, and generate alerts from monitored endpoints.

Verifying that Wazuh is installed:
![alt text](wazuh_showing_installed.png)


## 1.5 TheHive Installation
Unlike Wazuh, installing TheHive requires more steps.

For the brevity of this document, refer to TheHive website for instructions for steps 1.5.1 - 1.5.4. 

### 1.5.1 Install required dependencies

### 1.5.2 Set up the Java virtual machine (JVM)

### 1.5.3 Install and configure Apache Cassandra

### 1.5.4 Install and configure Elasticsearch

### 1.5.5 Install and configure TheHive
With the previous pre-requisites being installed, we can now download the installation packages for TheHive and finally  install it.

To Install TheHive, use the command:

    sudo apt-get install /tmp/thehive_5.7.3-1_all.deb

![alt text](thehive_installed.png)

Verifying that TheHive, Elasticsearch, and Cassandra are installed:
![alt text](thehive_showing_installed-1.png)

## 1.6 Logging into Wazuh and TheHive
Now that Wazuh and TheHive have been installed, we can try logging into them using our browser.

To login to our Wazuh Server using our web broswer we can simply type in

    https://[WAZUH-SERVER-PUBLIC-IP]

Which for our case is https://20.205.120.231/

However, upon trying this, we can see that we get a timeout error. Meaning our browser cannot connect to our Wazuh Server
![alt text](wazuh_timeout.png)

To login to our TheHive on the other hand, we can type in

    http://[THEHIVE-SERVER-PUBLIC-IP]:9000

However, like our Wazuh Server, this is giving us a timeout error.
![alt text](thehive_timeout.png)

So we must troubleshoot to figure out what is causing this and fix the issue.

### 1.6.1 Troubleshooting Wazuh and TheHive Servers
The first thing to check if there is a Firewall set up in our virtual network within Azure.

To do this we can simply go to Azure Portal -> Virtual Networks -> [virtual-network] -> Settings -> Firewall

For both our Vnets, there are no firewalls that exist. So we can eliminate this potential cause.

![alt text](wazuh_firewall.png)

![alt text](thehive_firewall.png) 