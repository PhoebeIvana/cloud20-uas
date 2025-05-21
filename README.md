# Cloud Computing Final Project Install and Configure Apache Cloudstack

## Group 20:

- George William Thomas Gonanta - 2206024253
- Phoebe Ivana - 2206820320
- Kania Aidilla Firka - 2206062983
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
  - [MySQL Installation](#mysql-installation)
- [Step 6: Creating a Zone](#creating-a-zone)
  - [Fill the Zone Type Section](#1️-zone-type)
  - [Fill the Core Zone Type Section](#2️-core-zone-type)
  - [Fill the Zone Details Section](#3-zone-details)
  - [Fill the Network Section](#4️-network)
    - [Defining its Physical Network](#physical-network-default)
    - [Defining its Public Traffic](#public-traffic)
  - [Adding the Resources](#5-add-resources)
    - [Fill the Pod Configuration](#pod-configuration)
    - [Fill the VLAN Range](#vlan-range-for-guest-traffic)
    - [Fill the Cluster Configuration](#cluster-configuration)
    - [Add A Host](#add-host)
    - [Set the Primary Storage Configuration](#primary-storage-configuration)
    - [Set the Secondary Storage Configuration](#secondary-storage-configuration)
- [Step 7: Registering an ISO](#registering-an-iso)
- [Registering an ISO](#registering-an-iso)
- [Step 8: Creating an Instance](#creating-an-instance)
  - [Go to Instances, then Click Add Instance +](#1-go-to-instances-then-click-add-instance-)
  - [Select a Deployment Infrastructure](#2-select-a-deployment-infrastructure)
  - [Select a Template/ISO](#3-select-a-templateiso)
  - [Select the Computing Offering and Disk Size](#4-select-the-computing-offering-and-disk-size)
  - [Select the Network](#5-select-the-network)
  - [Launch](#6-launch)
- [Step 9: Configure the VM's Network](#configure-the-vms-network)
- [Step 10: Setup the Port Forwarding](#setup-the-port-forwarding)
- [Step 11: Setup the Firewall](#setup-the-firewall)
- [Output](#output)

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

---

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

# Using KVM Hypervisor to Set Up a CloudStack Host

The procedures for configuring a CloudStack host with KVM (Kernel-based Virtual Machine) as the hypervisor are described in this document. In order to guarantee compatibility with the CloudStack Management Server, it covers the installation of necessary packages, network setup, access configuration, and system modifications.

---

## Set up CloudStack Agent and KVM

```bash
apt-get install qemu-kvm cloudstack-agent -y
```

---

## Modifying Configuration Documents

### Allow VNC to Hear on Every Interface

```bash
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf
```

This command modifies the `qemu.conf` file by adding an active line that instructs it to listen on all IP addresses in place of the commented line for `vnc_listen`.

---

### Permit remote connections to libvirtd

To enable libvirtd to listen for remote connections on Ubuntu 22.04, change its default settings:

```bash
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

This command produces a backup with the `.bak` extension and adds the `--listen` flag to the line that begins with `LIBVIRTD_ARGS=`.

---

## Add libvirtd's TCP Listening Settings

```bash
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

The `libvirtd.conf` file is configured by these instructions to:

- Turn off TLS connections.
- Make TCP connections available.
- TCP port should be set to `16509`.
- Switch off the mDNS ads.
- Turn off TCP authentication.

---

## Mask Unused Sockets and Restart libvirtd

```bash
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

Unwanted libvirtd socket units are kept from launching by masking. All new configuration modifications are applied when the service is restarted.

---

## Docker with Bridged Networking System Configurations

```bash
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

These configurations make guarantee that arptables and iptables don't filter bridged network packets needlessly.

---

## Create an Individual Host UUID

```bash
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

This guarantees that every host has a unique identity, which is crucial in settings where there are several virtual hosts.

---

## Set up firewall rules in iptables

```bash
NETWORK=192.168.1.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 3128 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 3128 -j ACCEPT
```

For communication within the designated network range (`192.168.1.0/24`), these rules open the necessary ports. They are particularly crucial for administration interfaces, CloudStack, and NFS services.

Make the guidelines enduring:

```bash
apt-get install iptables-persistent
```

When asked to preserve the rules, select "yes."

---

## Turn off libvirt AppArmor Profiles

```bash
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

To avoid limitations that can disrupt virtualization services, these instructions turn off AppArmor profiles for the `libvirtd` daemon and its companion tools.

---

## Start Apache CloudStack Management Server

```bash
cloudstack-setup-management
systemctl status cloudstack-management
tail -f /var/log/cloudstack/management/management-server.log
```

The CloudStack management service is now configured and operational. To live watch the log and confirm when the system is completely functional, use `tail -f`.

---

## Interface for Access Management

After the service has finished, launch a web browser and go to:

```
http://<YOUR_IP_ADDRESS>:8080
```

Example:

```
http://100.120.116.80:8080
```

## The CloudStack dashboard should become accessible at this stage.

![Image](https://github.com/user-attachments/assets/38d0c8d4-f6e4-4ca1-a36a-14c5ce86fe8b)

# Creating A Zone

After a successful installation and login, this tutorial explains how to create a new zone in Apache CloudStack. This repository has screenshots for every significant milestone.

---

## Step-by-Step Guide:

### 1️. Zone Type

The data center's architecture is determined by the zone type.

![Image](https://github.com/user-attachments/assets/6e00b2af-5d35-46cf-be23-8eaeed7ab9ca)

- Select the type of zone:  
  ✅ **Core**  
  _(The alternative is Edge, which is for easier setups near the user.)_

---

### 2️. Core Zone Type

How networking will be managed in the zone is determined by network model.

![Image](https://github.com/user-attachments/assets/f373538c-709d-4179-be59-62b26b6d2d23)

- Select the network model:  
  ✅ **Advanced**
  _(The other option is Basic; for greater control and flexibility in this configuration, we pick Advanced.)_

---

### 3. Zone Details

Zone details specify the zone's fundamental identification and networking configuration.

![Image](https://github.com/user-attachments/assets/2fc9c721-4b88-47c4-99ce-b81172a06eab)

- **Zone Name**: `zone-20`
- **IPv4 DNS 1**: `8.8.8.8`
- **Internal DNS 1**: `192.168.1.1` (Router's IP)
- **Guest CIDR Address**: Use default (e.g., `10.1.1.0/24`)
- **Hypervisor**: `KVM`

---

### 4️. Network

#### Physical Network (Default)

Since this solution does not require a custom physical layout or hardware-specific settings, the physical network setting is kept at its default configuration.

![Image](https://github.com/user-attachments/assets/8a7a5ab0-3cc1-4336-94ec-8de48b497716)

#### Public Traffic

The way external (public) network access is managed is determined by the public traffic settings.

![Image](https://github.com/user-attachments/assets/84bad439-e77e-429f-a55d-0de6d39d3b3f)

- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **Start IP**: `192.168.1.231`
- **End IP**: `192.168.1.235`
- **VLAN**: Leave empty (untagged)

---

### 5. Add Resources

#### Pod Configuration

A logical collection of hosts and primary storage that are connected to the same Layer 2 switch and subnet is called a pod.

![Image](https://github.com/user-attachments/assets/f72e85b1-39ed-4467-a1be-9ac11d87704b)

- **Pod Name**: `pod-20`
- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **IP Range**: `192.168.1.236 - 192.168.1.240`

#### VLAN Range (for Guest Traffic)

![Image](https://github.com/user-attachments/assets/afe3c196-74ef-4e83-a311-12d83f638c78)

The range of VLAN IDs that can be utilized to isolate network traffic from guests is specified by the VLAN range.
To keep traffic separate and boost security, each guest network can be given its own VLAN ID.

- **VLAN Range**: `3300-3399`

#### Cluster Configuration

A cluster is a collection of hosts that share access to primary storage and run the same hypervisor.

![Image](https://github.com/user-attachments/assets/5c593c93-f912-414c-8284-a65c4fb3a520)

- **Cluster Name**: `cluster-20`

#### Add Host

The actual device that powers the virtual computers is called a host.

![Image](https://github.com/user-attachments/assets/e498c962-3e20-4142-bf18-c20e8b65a890)

- **Hostname**: `192.168.1.219` (Server's IP)
  > **Note**: Enter the host's IP address and the username (e.g., `root`).  
  > The password field is required but hidden for security reasons.  
  > Make sure to enter the correct root password even though it won’t be displayed.

#### Primary Storage Configuration

All of the disk volumes for virtual machines (VMs) are kept in primary storage. Every host in the cluster needs to be able to access it.

![Image](https://github.com/user-attachments/assets/78cce944-a10c-43d2-b2fa-ddee27ddc48f)

- **Name**: `primary-20`
- **Scope**: Zone
- **Protocol**: `nfs`
- **Server**: `192.168.1.219`
- **Path**: `/report/primary`
- **Provider**: `DefaultPrimary`

#### Secondary Storage Configuration

ISO images, VM disk volume snapshots, and VM templates are stored on secondary storage. Additionally, all hosts in the zone must be able to access it.

![Image](https://github.com/user-attachments/assets/974a4d4f-92b7-4de7-ac35-5b9332a958d8)

- **Provider**: `NFS`
- **Name**: `secondary-20`
- **Server**: `192.168.1.219`
- **Path**: `/report/secondary`

---

### 6️. Launch

When the zone is launched, the configuration is complete and all specified resources—network, hosts, storage, etc.—become operational.

- Examine the setup and select **Launch Zone**.
- Await the initialization of the host, storage, and network resources.

The zone is activated and ready for virtual machine deployment after it is launched.

---

# Registering an ISO

The next step is to register an ISO when the zone has been successfully created and enabled. Virtual machines (VMs) will thereafter be installed in the zone we just created using this ISO.

## Navigation Path:

`Image ➝ ISO ➝ Register ISO`

## Step-by-Step Guide:

![Image](https://github.com/user-attachments/assets/e47061eb-9f5c-4535-a597-658277ab7400)

1. **URL**  
   Enter the ISO's direct download link:  
   `https://releases.ubuntu.com/noble/ubuntu-24.04.2-live-server-amd64.iso`

2. **Name**  
   Give the ISO a meaningful name: `ubuntu-server`

3. **Description**  
   Give a brief explanation: `Ubuntu Server`

4. **Direct Download**  
   This option should remain **disabled** (gray toggle).

   > CloudStack will automatically retrieve the ISO from the URL if it is enabled.

5. **Zone**  
   Choose the already created zone: `zone-20`

6. **Bootable**  
   Turn on this feature (blue toggle).
   > In order to install virtual machines (VMs), this designates the ISO as bootable.
7. **OS Type**  
   Choose the proper operating system: `Other Ubuntu (64-bit)`

---

## Note:

Unless certain advanced customizations are needed, all other fields not specifically listed above should be **left with their default values**.

---

# Creating an Instance

**The purpose** of creating an instance is to launch a new vVM on top of our infrastructure set up. The instance will utilize our network and storage resources we've already set up in the zone so that we can run services on a cloud.

## Steps

### 1. Go to `Instances`, then Click `Add Instance +`

### 2. Select a Deployment Infrastructure

Choose:

- **Zone** with the one that we have already created, which is `zone-20`.
- **Pod** with the one that we have already created, which is `pod-20`.
- **Cluster** with the one that we have already created, which is `cluster-20`.
- **Host** with the one that we have already created, which is `cloud-20`.
  ![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814564992.jpg?updatedAt=1747851710899)

### 3. Select a Template/ISO

On this step, make sure that we select the ISO form (instead of the `Template` form) that we have already registered before, which is `ubuntu-server`. Below is the following example to fill the `ISO` template:

![image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814585222.jpg?updatedAt=1747852976921)

**Remember** to choose the `KVM` as the hypervisor since we have already used it for virtualization (was defined during the initial setup process).

### 4. Select the Computing Offering and Disk Size

On this step, we could select the amount of computing resources and disk space that we want to allocate for the VM. We choose the `Medium` compute offering and `Medium` disk offering (note: there is no specific reason why we choose `Medium`, we just want to play safe, however the `small` option is usually enough for an OS like Ubuntu Server).
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814628431.jpg?updatedAt=1747853722626)

### 5. Select the Network

On this step, choose the network that we have created before during the zone setup, which is `isolated-20`. Remember to fill the `Name` field only and everything else by default.
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814686641.jpg?updatedAt=1747856034685)

### 6. Launch

Last, click the `Launch Instance` button, and the result will look something like this.
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814722118.jpg?updatedAt=1747856143405)

# Configure the VM's Network

After we're done with the previous step, next we should find the network that is used for the current VM, which is `isolated-20`. Then, go to the `Egress Rule` and define two rules:

- ICMP, set the destination CIDR to `0.0.0.0/0`, allowing our VM to send traffics to any IP on the internet.
- TCP, set the destination CIDR to `0.0.0.0/0`, allowing the VM to connect with any IP over TCP, such as SSH, HTTP, etc.
  ![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814829125.jpg?updatedAt=1747856879630)

# Setup the Port Forwarding

On this step, we want to allow external access towards the services inside our VM, therefore we should go to `Network`-> `Public IP Addresses` -> `Port Fowarding`. Here, we are adding a rule to forward all the TCP ports from public IP to VM's internal IP (for security purposes).
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814941649.jpg?updatedAt=1747857343440)

# Setup the Firewall

On this step, we want to allow SSH and HTTP/web server access, and therefore we defined the following rules:
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747815006688.jpg?updatedAt=1747858089238)

Both are using the same source CIDR (`0.0.0.0/0`), which means SSH and web server access to the VM is allowed from every external IP addresses.

# Output

![Image](https://ik.imagekit.io/livxgezjhg/Screenshot%202025-05-22%20031155.jpg?updatedAt=1747858327481)
We successfully access and ran the web server service.
