# Active Directory Project

# What is Active Directory?

- A database contains users, computers, groups
- In order to use AD a server must install service Active Directory Domain Services (ADDS)
- Then the server promoted to a domain controller
- The DC grant us the capability to perform authentication using a protocol called kerberos and authorization

### With AD DS database

- will contain objects such as users computers, groups
- This object contains attributes which is information about the object

> For example: Object - User = BOB Attribute, Firstname = BOB Last Name - Smith
> 

# Introduction

This homelab gives a hands-on exeprience in IT administration and cybersecurity by setting up Active Directory environment, Ingesting logs into SIEM (Splunk) monitoring events, and using tools like Sysmon to detect malicious activity and create alerts. As well as red team launching attacks via kali linux, Atmoic red team to simulate real-world threats and gaining practical experience in managing domains, users

## Machines Required

Windows 10 - Target Machine

Kali Linux - Attacker

Windows Server 2022 - Active Directory

Ubunut Server - Splunk 

# Network Diagram

![2026-04-05 23_39_11-Arc.png](Active%20Directory%20Project/2026-04-05_23_39_11-Arc.png)

## Network Adapter

- Go to Network tab of virtual box > create a separate NAT adapter for machines

![image.png](Active%20Directory%20Project/image.png)

# Installation of Ubuntu Server

- Click on new button to create virtual machine
- Give it a name, select vm folder
- Allocate base memory & hard disk space
- Go to Settings > Network > Select the network adapter we created earlier
- Boot up machine it’ll ask to provide the ISO file select the ISO from the dropdown
- Go with the default setup process
- Set username password

![2026-04-05 23_40_31-Splunk [Running] - Oracle VirtualBox.png](Active%20Directory%20Project/2026-04-05_23_40_31-Splunk_Running_-_Oracle_VirtualBox.png)

![2026-04-05 23_46_05-Splunk [Running] - Oracle VirtualBox.png](Active%20Directory%20Project/2026-04-05_23_46_05-Splunk_Running_-_Oracle_VirtualBox.png)

- **Update the server**

`sudo apt-get update && sudo apt-get upgrade -y`

- **Install virutalbox-guest-addition (To use shared folder between hosts)**

virtualbox-guest-additions is a collection of devices drivers and system application designed to be installed inside VM to improve performance and usability 

![2026-04-06 00_02_54-Splunk (First Install) [Running] - Oracle VirtualBox.png](Active%20Directory%20Project/2026-04-06_00_02_54-Splunk_(First_Install)_Running_-_Oracle_VirtualBox.png)

- **Install virtualbox-guest-utilis**

`sudo apt-get install virutalbox-guest-utlis`

- Assign static ip to server
- Add share folder
- Add user in vboxsf
- Mount folder under vboxsf
- Install splunk & accept license &enable startup at up boot &
- Install universal forwarder

# IP configuration

Navigate to `/etc/netplan/` open the file `50-cloud-init.yaml` with nano to update the IP

`sudo nano /etc/netplan/50-cloud-init.yaml`

![image.png](Active%20Directory%20Project/image%201.png)

![image.png](Active%20Directory%20Project/image%202.png)

Apply the configuration `sudo netplan apply`

![image.png](Active%20Directory%20Project/image%203.png)

As we can see we have successfully change the IP

# Shared Folder

To add shared folder to VM. Navigate to devices > Shared Folders

![image.png](Active%20Directory%20Project/image%204.png)

![image.png](Active%20Directory%20Project/image%205.png)

![image.png](Active%20Directory%20Project/image%206.png)

Click **Add new shared folder** > Select the folder you want to share

Check Read-only, Auto-mount, Make Global

![image.png](Active%20Directory%20Project/image%207.png)

After adding folder add user to vboxsf (Virtualbox shared fodler)

`sudo adduser splunk vboxsf`
adds your current Linux guest user to the `vboxsf` group, allowing you to access VirtualBox shared folders without `root` permissions

Make a directory `mkdir folder`

`sudo mount -t vboxsf -o uid=1000,gid=1000 newfolder folder/`

Used within a **Linux Guest virtual machine** to manually mount a folder shared from the **Host machine**

mount - used to mount files from host to VM, 

-t type - “Mount a shared folder provided by VirtualBox”

-o set ownership `uid=1000` → your user ID `guid=1000` → your group ID

new_folder - shared folder

folder - point to mount

- **Now copy the splunk / splunk universal forwarder installation files to the mounted folder**

# Installation of splunk / Universal Forwarder

`sudo dpkg -i splunk-installation-file`

![image.png](Active%20Directory%20Project/image%208.png)

- The default folder is `/opt`

**Initialize and Start Splunk**

- `sudo /opt/splunk/bin/splunk start --accept-license`
- Add username and password for splunk web interface

![image.png](Active%20Directory%20Project/image%209.png)

**Enable Boot Start**

- `sudo /opt/spunk/bin/splunk enable boot-start -user splunk`

**Install universal forwarder:**

`sudo dpkg -i splunkforwarderfile` 

![image.png](Active%20Directory%20Project/image%2010.png)

**Changing ownership**

`sudo chown  -R splunk:splunk /opt/splunkforwarder/`

**Start Splunk and Accept License**

`sudo /opt/splunkforwareder/bin/splunk start —accept-license`

**Change the port to 9997 while installation as it’s the default port to forward data to splunk**

**Enable Boot-Start**Configure the forwarder to start automatically on system boot:

`sudo /opt/splunkforwarder/bin/splunk enable boot-start`

**Configure Forwarding**Set up the forwarder to send data to your Splunk Indexer (receiver):

`sudo /opt/splunkforwarder/bin/splunk add forward-server <indexer_ip_address>:9997`

# Installing Windows Server

- Settings > Network Tab > Change the adapter we created earlier > OK

Start the Machine 

- Next > Accept license terms > Custom install > Select the drive > Next, **Set password for administrator account,** Rename the PC, Reboot then install guest-addition-iso, Create shared folder, Install splunk forwarder set port, username, password,
- Add users to AD, Navigate to server manager tools > AD users and computers > under any units > add user and enter the necessary information

![2026-04-07 15_03_38-Peek.png](Active%20Directory%20Project/2026-04-07_15_03_38-Peek.png)

![image.png](Active%20Directory%20Project/image%2011.png)

# IP Configuration

- Open Network & Internet Settings
- Change Adapter Options
- Adapter Properties > IPv4 properties > use the following IP address > Assign IP

## Configuring AD-DC

![image.png](Active%20Directory%20Project/image%2012.png)

![image.png](Active%20Directory%20Project/image%2013.png)

**Click Add Roles and Features > Next** 

**Installation Type > Role-Based or feature-based installation > Next**

**Server Selection > Default selection > Next**

**Server Roles > Active Directory Domain System > Next > Next > Install**

![image.png](Active%20Directory%20Project/image%2014.png)

![image.png](Active%20Directory%20Project/image%2015.png)

# Promoting to Domain Controller

- Click on the Flag icon

![image.png](Active%20Directory%20Project/image%2016.png)

![image.png](Active%20Directory%20Project/image%2017.png)

- Promote this server to a domain controller
- Select `Add a new forest` & Enter root domain name
- Enter Password
- Click Install
- Select the VM  ISO and Boot

![image.png](Active%20Directory%20Project/image%2018.png)

![image.png](Active%20Directory%20Project/image%2019.png)

# Installing Windows 10

- Click **Install Now**.
- Select Windows 10 edition.
- Choose **Custom: Install Windows only**.
- Follow on-screen steps to set up user account and preferences.
- After Installation Rename PC name,
- Click **Devices → Insert Guest Additions CD Image**.
- Restart the VM.

# Network Configuration

- Open **Control Panel**.
- Navigate to **Network and Sharing Center**.
- Click on **Change adapter settings**.
- Right-click on the active network adapter and select **Properties**.
- Select **Internet Protocol Version 4 (IPv4)** and click **Properties**.
- Choose **Use the following IP address**.
- IP Address, Subnet Mask, Default Gateway:
- Click **OK** to save changes.

![image.png](Active%20Directory%20Project/image%2020.png)

# Installing Sysmon

- Copy the sysmon zip file and [sysmon config file](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml) to desktop and open the directory in powershell, execute commands

`Set-ExecutionPolicy unrestricted Currentuser` `./sysmon.exe -i sysmonconfig.xml` and follow up - -the wizard, And edit the config file inputs.confi under `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf` to monitor what and how. Here’s a -conf file by [MYDFIR](https://github.com/MyDFIR/Active-Directory-Project) you can go with this. Repeat this same steps on windows AD

# Splunk Configuration WIN-10 & WIN-AD

- Install `splunk universal forwarder` from shared folder, set username & password, set default port `9997`

![image.png](Active%20Directory%20Project/image%2021.png)

![2026-04-12 14_07_40-WIN-10 (First install) [Running] - Oracle VirtualBox.png](Active%20Directory%20Project/2026-04-12_14_07_40-WIN-10_(First_install)_Running_-_Oracle_VirtualBox.png)

- Access the spunk web-interface from windows 10, Navigate to settings > Forwarding and receiving > Under Receive data click Add new and add the port we set earlier `9997` and

![image.png](Active%20Directory%20Project/image%2022.png)

![image.png](Active%20Directory%20Project/image%2023.png)

![image.png](Active%20Directory%20Project/image%2024.png)

- Next we need to create index head to settings > indexes > new index > give it a name > save
- If properly configured splunkforwarder, we’ll see 2 host

![image.png](Active%20Directory%20Project/image%2025.png)

![image.png](Active%20Directory%20Project/image%2026.png)

![image.png](Active%20Directory%20Project/image%2027.png)

# Performing Attacks with Kali Linux

- Change the IP address to static under the network and connections settings
- We’ll perform a rdp brute force attack with hydra to user jputh

![image.png](Active%20Directory%20Project/image%2028.png)

- As we can see event id 4625

![image.png](Active%20Directory%20Project/image%2029.png)

![image.png](Active%20Directory%20Project/image%2030.png)

# Atomic Red Team

- Add C drive to exclusion in windows Security so it wont remove any files while scanning

![image.png](Active%20Directory%20Project/image%2031.png)

![image.png](Active%20Directory%20Project/image%2032.png)

`set-executionpolicy unrestricted currentuser`

`.\install-atmocredteam.ps1` > Navigate to atmoics direcotry \ `Invoke-AtomicTest T1136.001`

- Mitre `T1136.001`: Adversaries may create a local account to maintain access to victim systems. Local accounts are those configured by an organization for use by users, remote support, services, or for administration on a single system or service.

![image.png](Active%20Directory%20Project/image%2033.png)

![image.png](Active%20Directory%20Project/image%2034.png)

![image.png](Active%20Directory%20Project/image%2035.png)