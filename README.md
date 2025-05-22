# FINAL PROJECT: CLOUD COMPUTING

![Image](https://ik.imagekit.io/livxgezjhg/logo%20ui.png?updatedAt=1747858930514)

## Group 20:

- George William Thomas Gonanta - 2206024253
- Phoebe Ivana - 2206820320
- Kania Aidilla Firka - 2206062983
- Darren Nathanael Boentara - 2206059490

## List of Contents

---

- [Introduction](#introduction)  
  - [What is Apache Cloudstack](#what-is-apache-cloudstack)  
  - [Why Choose Apache Cloudstack](#why-choose-apache-cloudstack)  
  - [Dependencies](#depedencies)  
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
- [Step 6: Using KVM Hypervisor to Set Up a CloudStack Host](#step-6-using-kvm-hypervisor-to-set-up-a-cloudstack-host)  
  - [Set up CloudStack Agent and KVM](#set-up-cloudstack-agent-and-kvm)  
  - [Modify Configuration Files](#modifying-configuration-documents)  
    - [Allow VNC to Listen on All Interfaces](#allow-vnc-to-hear-on-every-interface)  
    - [Permit Remote libvirtd Connections](#permit-remote-connections-to-libvirtd)  
    - [Enable libvirtd TCP Settings](#add-libvirtds-tcp-listening-settings)  
  - [Mask Unused libvirtd Sockets](#mask-unused-sockets-and-restart-libvirtd)  
  - [Docker with Bridged Networking System Configurations](#docker-with-bridged-networking-system-configurations)  
  - [Create a Unique Host UUID](#create-an-individual-host-uuid)  
  - [Configure iptables Rules](#set-up-firewall-rules-in-iptables)  
  - [Disable AppArmor for libvirtd](#turn-off-libvirt-apparmor-profiles)  
  - [Start Apache CloudStack Management Server](#start-apache-cloudstack-management-server)  
  - [Access Management Interface](#interface-for-access-management)  
- [Step 7: Creating a Zone](#step-7-creating-a-zone)  
  - [Zone Type](#1️-zone-type)  
  - [Core Zone Type](#2️-core-zone-type)  
  - [Zone Details](#3-zone-details)  
  - [Network](#4️-network)  
    - [Physical Network](#physical-network-default)  
    - [Public Traffic](#public-traffic)  
  - [Add Resources](#5-add-resources)  
    - [Pod Configuration](#pod-configuration)  
    - [VLAN Range](#vlan-range-for-guest-traffic)  
    - [Cluster Configuration](#cluster-configuration)  
    - [Add Host](#add-host)  
    - [Primary Storage Configuration](#primary-storage-configuration)  
    - [Secondary Storage Configuration](#secondary-storage-configuration)  
  - [Launch](#6️-launch)  
- [Step 8: Registering an ISO](#step-8-registering-an-iso)  
- [Step 9: Creating an Instance](#step-9-creating-an-instance)  
  - [Add Instance](#1-go-to-instances-then-click-add-instance-)  
  - [Deployment Infrastructure](#2-select-a-deployment-infrastructure)  
  - [Template/ISO](#3-select-a-templateiso)  
  - [Computing Offering](#4-select-the-computing-offering-and-disk-size)  
  - [Network](#5-select-the-network)  
  - [Launch](#6-launch)  
- [Step 10: Configure the VM's Network](#step-10-configure-the-vms-network)  
- [Step 11: Setup the Port Forwarding](#step-11-setup-the-port-forwarding)  
- [Step 12: Setup the Firewall](#step-12-setup-the-firewall)  
- [Output](#output)

# Introduction

### What is Apache Cloudstack

Apache CloudStack is an open source Infrastructure as a Service (IaaS) cloud computing platform that is made to deploy and manage a massive virtual machine networks. It is one of the most popular option for developing a private, public, and hybrid cloud environments because of its scalability, dependability, and user-friendliness.

### Why Choose Apache Cloudstack?

The purpose of Apache CloudStack is to make cloud infrastructure setup easier. How? through a single interface. Basically, it simplify users to manage resources by abstracting away the complexity of networking, storage, and physical hardware.

### Depedencies

- MySQL
  Apache CloudStack uses MySQL for its primary database in storing all the configuration data, operational metadata, and also system state. The stored data also includes the user accounts, virtual machine details, logs, network configuratios, storage mappings, and job histories.

- KVM (Kernel-based Virtual Machine)
  KVM (Kernel-based Virtual Machine) is a virtualization technology integrated for the Linux kernel that allow a physical server to operate many virtual machines (VMs) at once. These VMs can have its own operating system and resources. KVM is one of the hypervisors that Apache CloudStack officially supports. This is done because of its stability, open-sorce status, and native Linux integration. KVM is used by CloudStack to build and manage virtual machines on physical computing nodes (hosts), within the cloud infrastruture.

# Step 1: Ubuntu 24.04 LTS Installation

a. Change the SATA operation (driver to AHCI).

b. Download an Ubuntu Image from https://ubuntu.com/download/desktop.

c. Install Ubuntu Desktop by writing the downloaded ISO to a USB stick (may follow this tutorial: https://etcher.balena.io/)

d. Insert the USB flash drive to the laptop that we are using, then boot or restart the device. It will automatically recognise the installation media.

e. Do the usual installation setup, then create the login details.

f. The installation process is done.

# Step 2: Network Configuration

### Modify The Network Configuration File

First, modify the network configuration file in the `/netplan` directory. Change the directory to netplan and open the file using the following command:

```
cd /etc/netplan
sudo nano ./0*.yaml
```

The next step is to edit the YAML as follows:

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

After modifying the YAML file, apply the configuration with the following command:

```
netplan generate    # This will generate a configuration files from /etc/netplan/*.yaml
netplan apply       # Apply it
reboot              # Restart the system, just to make sure that the changes is saved already
```

# Step 3: Monitoring Tools Installation

On this step, log in as the root user to ensure that we have the necessary administrative privileges.

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

- **htop** is a monitoring tools of CPU and process usage.
- **duf** is amonitoring tools of disk usage.
- **lynx** is a text-based web browser.
- **bridge-utils** is a tools for managing network bridges.

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

Make sure **Password** is changed to the **Ubuntu's root password** and **Server_IP** is changed to the **IP Address of the Ubuntu Machine**

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

# Step 6: Using KVM Hypervisor to Set Up a CloudStack Host

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

# Step 7: Creating A Zone

Following a successful installation and login, this tutorial describes creating a new zone in Apache CloudStack. This repository contains screenshots for each significant achievement.

---

## Step-by-Step Guide:

### 1️. Zone Type

The type of zone determines the data center’s architecture.

![Image](https://github.com/user-attachments/assets/6e00b2af-5d35-46cf-be23-8eaeed7ab9ca)

Choose the following one core types of zone:
  ✅ **Core**  
  _(Edge is the alternative which is meant for simpler configurations closer to the end user)_

---

### 2️. Core Zone Type

Network model defines how networking will be done in the zone.

![Image](https://github.com/user-attachments/assets/f373538c-709d-4179-be59-62b26b6d2d23)

Choose the network model:
  ✅ **Advanced**
  _(The other option is Basic. Considering the need to implement control and flexibility into this configuration, it is chosen Advanced.)_

---

### 3. Zone Details

Zone details capture the fundamental identification of the zone and its corresponding networking configuration.

![Image](https://github.com/user-attachments/assets/2fc9c721-4b88-47c4-99ce-b81172a06eab)

- **Zone Name**: `zone-20`
- **IPv4 DNS 1**: `8.8.8.8`
- **Internal DNS 1**: `192.168.1.1` (Router's IP)
- **Guest CIDR Address**: Use default (e.g., `10.1.1.0/24`)
- **Hypervisor**: `KVM`

---

### 4️. Network

#### Physical Network (Default)

As no custom layout or hardware specific features are needed, the setting is maintained at default.  

![Image](https://github.com/user-attachments/assets/8a7a5ab0-3cc1-4336-94ec-8de48b497716)

#### Public Traffic

Controls on how public network access is provided is configured by the public traffic settings.

![Image](https://github.com/user-attachments/assets/84bad439-e77e-429f-a55d-0de6d39d3b3f)

- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **Start IP**: `192.168.1.231`
- **End IP**: `192.168.1.235`
- **VLAN**: Leave empty (untagged)

---

### 5. Add Resources

#### Pod Configuration

Pod is a term that refers to a logical group of hosts and primary storage that are interconnected with the same Layer 2 switch and subnet.  

![Image](https://github.com/user-attachments/assets/f72e85b1-39ed-4467-a1be-9ac11d87704b)

- **Pod Name**: `pod-20`
- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **IP Range**: `192.168.1.236 - 192.168.1.240`

#### VLAN Range (for Guest Traffic)

![Image](https://github.com/user-attachments/assets/afe3c196-74ef-4e83-a311-12d83f638c78)

The VLAN range describes the limit of VLAN ID’s that can be used to merge network traffic from guests. To maintain segregation and increase security, each guest network can be assigned a separate VLAN ID.

- **VLAN Range**: `3300-3399`

#### Cluster Configuration

Cluster is defined as a set of hosts having common access to primary storage and running the same hypervisor.

![Image](https://github.com/user-attachments/assets/5c593c93-f912-414c-8284-a65c4fb3a520)

- **Cluster Name**: `cluster-20`

#### Add Host

Host is the underlying physical system that arms the virtual machines with computing resources.

![Image](https://github.com/user-attachments/assets/e498c962-3e20-4142-bf18-c20e8b65a890)

- **Hostname**: `192.168.1.219` (Server's IP)
  > **Note**: For the host use the IP address and username set to root, for example.  
  > The password is optional and invisible due to security norms. 
  > Though the password is not visible, the correct root password should still be entered.

#### Primary Storage Configuration

Virtual Machines (VMs) have their disk volumes stored in primary storage. Each host in the cluster must have access to it.

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

# Step 8: Registering an ISO

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

# Step 9: Creating an Instance

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

# Step 10: Configure the VM's Network

After we're done with the previous step, next we should find the network that is used for the current VM, which is `isolated-20`. Then, go to the `Egress Rule` and define two rules:

- ICMP, set the destination CIDR to `0.0.0.0/0`, allowing our VM to send traffics to any IP on the internet.
- TCP, set the destination CIDR to `0.0.0.0/0`, allowing the VM to connect with any IP over TCP, such as SSH, HTTP, etc.
  ![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814829125.jpg?updatedAt=1747856879630)

# Step 11: Setup the Port Forwarding

On this step, we want to allow external access towards the services inside our VM, therefore we should go to `Network`-> `Public IP Addresses` -> `Port Fowarding`. Here, we are adding a rule to forward all the TCP ports from public IP to VM's internal IP (for security purposes).
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747814941649.jpg?updatedAt=1747857343440)

# Step 12: Setup the Firewall

On this step, we want to allow SSH and HTTP/web server access, and therefore we defined the following rules:
![Image](https://ik.imagekit.io/livxgezjhg/messageImage_1747815006688.jpg?updatedAt=1747858089238)

Both are using the same source CIDR (`0.0.0.0/0`), which means SSH and web server access to the VM is allowed from every external IP addresses.

# Output

![Image](https://ik.imagekit.io/livxgezjhg/Screenshot%202025-05-22%20031155.jpg?updatedAt=1747858327481)
We successfully access and ran the web server service.
