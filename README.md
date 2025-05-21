# Cloud Computing Final Project Install and Configure Apache Cloudstack

## Group 20:
- George William Thomas Gonanta - 2206
- Phoebe Ivana - 2206
- Kania Aidila Firka - 2206
- Darren Nathanael Boentara - 2206059490
---

- [Introduction](#introduction)
  - [What is Apache Cloudstack](#what-is-apache-cloudstack)
  - [Why Use Apache Cloudstack](#why-use-apache-cloudstack)
  - [Depedencies](#depedencies)
- [Step 1: Ubuntu 24.04 LTS Installation](#step-1-ubuntu-2404-lts-installation)
- [Step 2: Network Configuration](#step-2-network-configuration)
  - [Modify The Network Configuration File](#modify-the-network-configuration-file)
  - [Apply The Configuration](#apply-the-configuration)
- [Step 3: Monitoring Tools Installation](#step-3-monitoring-tools-installation)
  - [Update System Packages](#update-system-packages)
  - [Install Monitoring and Utility Tools](#install-monitoring-and-utility-tools)
- [Step 4: SSH Configuration](#step-4-ssh-configuration)
  - [Install OpenSSH Server](#install-openssh-server)
  - [Enable SSH root login](#enable-ssh-root-login)
- [Step 5: Apache CloudStack and MySQL Installation](#step-5-apache-cloudstack-and-mysql-installation)
  - [Apache CloudStack Installation](#apache-cloudstack-installation)
    - [Import Apache CloudStack Repositories Key](#import-apache-cloudstack-repositories-key)
    - [Install Apache CloudStack and MySQL Server](#install-apache-cloudstack-and-mysql-server)
  - [MySQL Installation](#mysql-installation)
    - [Configure MySQL](#configure-mysql)
    - [Configure NFS (Network File System) Server](#configure-nfs-network-file-system-server)

# Introduction
### What is Apache Cloudstack
Apache CloudStack is an open-source Infrastructure as a Service (IaaS) cloud computing platform designed to deploy and manage large networks of virtual machines. It is highly scalable, reliable, and easy to use, making it a popular choice for creating private, public, and hybrid cloud environments.

### Why Use Apache Cloudstack
Apache CloudStack is designed to simplify the process of setting up a cloud infrastructure. It abstracts the complexity of physical hardware, networking, and storage, enabling users to provision and manage resources easily through a unified interface.

### Depedencies
- MySQL
Apache CloudStack uses MySQL as its primary relational database system to store all configuration data, operational metadata, and system state. This includes user accounts, virtual machine details, network configurations, storage mappings, logs, and job histories.

- KVM (Kernel-based Virtual Machine)
KVM (Kernel-based Virtual Machine) is a virtualization technology built directly into the Linux kernel that allows a physical server to run multiple virtual machines (VMs) simultaneously, each with its own operating system and resources. KVM is one of the officially supported hypervisors by Apache Cloudtack and is commonly used due to its open-source nature, stability, and native integration with Linux. CloudStack uses KVM to create and manage virtual machines across physical compute nodes (also called hosts) in the cloud infrastructure.

# Step 1: Ubuntu 24.04 LTS Installation
a. Change the SATA operation (driver to AHCI).

b. Download an Ubuntu Image from https://ubuntu.com/download/desktop. 

c. Install Ubuntu Desktop by writing the downloaded ISO to a USB stick (you may follow this tutorial: https://etcher.balena.io/)

d. Insert the USB flash drive to the laptop that you are using, then boot or restart the device. It will automatically recognise the installation media. 

e. Do the usual installation setup, then create your login details.

f. The installation process is done.

# Step 2: Network Configuration
### Modify The Network Configuration File
The first thing you can do is to modify the network configuration file in the /netplan directory. Change the directory to netplan and open the file using this command.

```
cd /etc/netplan
sudo nano ./0*.yaml
```
Next, edit the YAML file like this.

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      dhcp6: false
      optional: true
  bridges:
    cloudbr0:
      addresses: [192.168.1.219/24] 
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp0s3]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```

### Apply The Configuration
After modifying the YAML file, apply the configuration with this command.

```
netplan generate    # Generate configuration files from /etc/netplan/*.yaml
netplan apply       # Apply the generated network configuration
reboot              # Restart the system to ensure changes take full effect
```

# Step 3: Monitoring Tools Installation
First, log in as the root user to ensure you have the necessary administrative privileges.

```
su -
```

### Update System Packages
Before installing any tools, update and upgrade the system to the latest package versions.

```
apt update -y
apt upgrade -y
```

### Install Monitoring and Utility Tools
To install essential tools for monitoring and managing system resources use this command.

```
apt install htop lynx duf -y
apt install bridge-utils
```

- **htop** provides a real-time, interactive view of CPU and process usage.
- **duf** offers a modern and user-friendly overview of disk usage.
- **lynx** is a lightweight, text-based web browser for use within the terminal.
- **bridge-utils** is required for managing network bridges, which are useful in virtualization and network configurations.

# Step 4: SSH Configuration

## Install OpenSSH Server

To allow other mahines to connect to the host through ssh, make sure that OpenSSH Server is installed
```bash
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh # to enable SSH to start on boot (optional)
```

## Enable SSH root login

To allow ssh connection as root, edit the sshd config file
```bash
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
sudo systemctl restart ssh # restart the SSH service to apply the changes
```

-----------------------------
# Step 5: Apache CloudStack and MySQL Installation

## Apache CloudStack Installation

### Import Apache CloudStack Repositories Key

To simplify the installation going forward, start a new shell as root
```bash
sudo -i
```
Prepare the necessary APT (Advanced Package Tool) key to install Apache CloudStack
```bash
mkdir -p /etc/apt/keyrings # the directory where the GPG key is stored
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null # downloads the GPG key >> converts it from ASCII-armored key (.asc) to binary format (.gpg) that is required by APT >> writes the binary key to cloudstack.gpg
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list # adds the Apache CloudStack APT repository to cloudstack.list
```

Update the APT package with the new Apache Cloudstack APT repositories
```bash
apt-get update -y
```

### Install Apache CloudStack and MySQL Server

Install the necessary tools
```bash
apt-get install cloudstack-management mysql-server
```

## MySQL Installation

### Configure MySQL

Open the MySQL Config
```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Paste these settings under the [mysqld] section
```text
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

Restart MySQL Service to apply the changes
```bash
systemctl restart mysql
```

Deploy a CloudStack database as root and create a user with username "cloud" and password "cloud"
```bash
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:<Pasword> -i <Server_IP>
```
Make sure **Password** is changed to your **Ubuntu's root password** and **Server_IP** is changed to the **IP Address of your Ubuntu Machine**

### Configure NFS (Network File System) Server

For Apache CloudStack to manage storage efficiently, NFS is used to create a primary and secondary storage
```bash
apt-get install nfs-kernel-server quota
```
```bash
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

Assign specific ports for NFS services (mountd, statd, quotad)
```bash
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```
