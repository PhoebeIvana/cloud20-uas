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

# Setting Up a CloudStack Host with KVM Hypervisor

This guide outlines the steps to configure a CloudStack host using KVM (Kernel-based Virtual Machine) as the hypervisor. It covers the installation of essential packages, network setup, access configuration, and system adjustments to ensure compatibility with the CloudStack Management Server.

---

## Install KVM and CloudStack Agent
```bash
apt-get install qemu-kvm cloudstack-agent -y
```

---

## Adjusting Configuration Files

### Enable VNC to Listen on All Interfaces

```bash
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf
```

This command updates the `qemu.conf` file by replacing the commented line for `vnc_listen` with an active line that sets it to listen on all IP addresses.

---

### Allow libvirtd to Accept Remote Connections

On Ubuntu 22.04, modify the default settings for libvirtd to allow it to listen for remote connections:

```bash
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

This command replaces the line starting with `LIBVIRTD_ARGS=` to include the `--listen` flag, and creates a backup with the `.bak` extension.

---

## Add TCP Listening Settings for libvirtd

```bash
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

These commands configure the `libvirtd.conf` file to:
- Disable TLS connections
- Enable TCP connections
- Set TCP port to `16509`
- Turn off mDNS advertisements
- Disable authentication over TCP

---

## Restart libvirtd and Mask Unused Sockets

```bash
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

Masking prevents unwanted libvirtd socket units from starting. Restarting the service applies all new configuration changes.

---

## System Settings for Docker and Bridged Networking

```bash
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

These settings ensure bridged network packets aren't unnecessarily filtered by arptables or iptables.

---

## Generate Unique Host UUID

```bash
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

This ensures that each host has a unique identifier, which is essential in environments with multiple virtual hosts.

---

## Configure iptables Firewall Rules

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

These rules open required ports for communication within the specified network range (`192.168.1.0/24`). They are especially important for services like NFS, CloudStack, and management interfaces.

Make the rules persistent:

```bash
apt-get install iptables-persistent
```

Answer `yes` when prompted to save the rules.

---

## Disable AppArmor Profiles for libvirt

```bash
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

These commands disable AppArmor profiles for the `libvirtd` daemon and its helper tools to prevent restrictions that could interfere with virtualization services.

---

## Start Apache CloudStack Management Server

```bash
cloudstack-setup-management
systemctl status cloudstack-management
tail -f /var/log/cloudstack/management/management-server.log
```

This sets up and starts the CloudStack management service. Use `tail -f` to live monitor the log and verify when the system is fully operational.

---

## Access Management Interface

Once the service is running, open a web browser and visit:

```
http://<YOUR_IP_ADDRESS>:8080
```

Example:

```
http://100.120.116.80/:8080
```

## The CloudStack dashboard should become accessible at this stage.
![Image](https://github.com/user-attachments/assets/38d0c8d4-f6e4-4ca1-a36a-14c5ce86fe8b)


# Creating A Zone

This guide documents the process of creating a new zone in Apache CloudStack after a successful installation and login. Screenshots are provided in this repository for each major step.

---

## Step-by-Step Guide:

### 1️. Zone Type
The zone type defines the architecture of the data center.

![Image](https://github.com/user-attachments/assets/6e00b2af-5d35-46cf-be23-8eaeed7ab9ca)

- Choose zone type:  
  ✅ **Core**  
  _(The other option is Edge; edge is for simplified setups close to the user)_

---

### 2️. Core Zone Type
Network model determines how networking will be managed in the zone.

![Image](https://github.com/user-attachments/assets/f373538c-709d-4179-be59-62b26b6d2d23)

- Select the network model:  
  ✅ **Advanced**
  _(The other option is Basic; for this setup we use Advanced for more control and flexibility)_

---

### 3. Zone Details
Zone details define the basic identity and networking settings of the zone.

![Image](https://github.com/user-attachments/assets/2fc9c721-4b88-47c4-99ce-b81172a06eab)

- **Zone Name**: `zone-20`
- **IPv4 DNS 1**: `8.8.8.8`
- **Internal DNS 1**: `192.168.1.1` (Router's IP)
- **Guest CIDR Address**: Use default (e.g., `10.1.1.0/24`)
- **Hypervisor**: `KVM`

---

### 4️. Network

#### Physical Network (Default)
The physical network setting is left at its default configuration because no custom physical layout or hardware-specific settings are required for this setup.

![Image](https://github.com/user-attachments/assets/8a7a5ab0-3cc1-4336-94ec-8de48b497716)

#### Public Traffic
Public traffic settings define how external (public) network access is handled.

![Image](https://github.com/user-attachments/assets/84bad439-e77e-429f-a55d-0de6d39d3b3f)

- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **Start IP**: `192.168.1.231`
- **End IP**: `192.168.1.235`
- **VLAN**: Leave empty (untagged)

---

### 5. Add Resources

#### Pod Configuration
A pod is a logical grouping of hosts and primary storage that share the same Layer 2 switch and subnet.

![Image](https://github.com/user-attachments/assets/f72e85b1-39ed-4467-a1be-9ac11d87704b)

- **Pod Name**: `pod-20`
- **Gateway**: `192.168.1.1`
- **Netmask**: `255.255.255.0`
- **IP Range**: `192.168.1.236 - 192.168.1.240`

#### VLAN Range (for Guest Traffic)

![Image](https://github.com/user-attachments/assets/afe3c196-74ef-4e83-a311-12d83f638c78)

The VLAN range defines the range of VLAN IDs that can be used for isolating guest network traffic.
Each guest network can be assigned a unique VLAN ID to maintain traffic separation and improve security.

- **VLAN Range**: `3300-3399`

#### Cluster Configuration
A cluster is a group of hosts that use the same hypervisor and share access to primary storage.

![Image](https://github.com/user-attachments/assets/5c593c93-f912-414c-8284-a65c4fb3a520)

- **Cluster Name**: `cluster-20`

#### Add Host
A host is a physical machine that runs the virtual machines.

![Image](https://github.com/user-attachments/assets/e498c962-3e20-4142-bf18-c20e8b65a890)

- **Hostname**: `192.168.1.219` (Server's IP)
> **Note**: Enter the host's IP address and the username (e.g., `root`).  
> The password field is required but hidden for security reasons.  
> Make sure to enter the correct root password even though it won’t be displayed.

#### Primary Storage Configuration
Primary storage is where all the disk volumes for virtual machines (VMs) are stored. It must be accessible to all hosts in the cluster.

![Image](https://github.com/user-attachments/assets/78cce944-a10c-43d2-b2fa-ddee27ddc48f)

- **Name**: `primary-20`
- **Scope**: Zone
- **Protocol**: `nfs`
- **Server**: `192.168.1.219`
- **Path**: `/report/primary`
- **Provider**: `DefaultPrimary`

#### Secondary Storage Configuration
Secondary storage is used to store VM templates, ISO images, and VM disk volume snapshots. It must also be accessible to all hosts in the zone.

![Image](https://github.com/user-attachments/assets/974a4d4f-92b7-4de7-ac35-5b9332a958d8)

- **Provider**: `NFS`
- **Name**: `secondary-20`
- **Server**: `192.168.1.219`
- **Path**: `/report/secondary`

---

### 6️. Launch
Launching the zone finalizes the configuration and brings all defined resources (network, hosts, storage, etc.) into an operational state.

- Review configuration
- Click **Launch Zone**
- Wait for resources (host, storage, network) to initialize

Once launched, the zone becomes active and available for deploying virtual machines.

---

# Registering an ISO

Once the zone has been successfully created and enabled, the next step is to register an ISO. This ISO will later be used to install virtual machines (VMs) in the zone we just set up.


## Navigation Path:
`Image ➝ ISO ➝ Register ISO`

## Step-by-Step Guide:

![Image](https://github.com/user-attachments/assets/e47061eb-9f5c-4535-a597-658277ab7400)

1. **URL**  
   Enter the direct download link for the ISO:  
   `https://releases.ubuntu.com/noble/ubuntu-24.04.2-live-server-amd64.iso`  

2. **Name**  
   Fill in a descriptive name for the ISO:  `ubuntu-server`

3. **Description**  
   Provide a brief description:  `Ubuntu Server`

4. **Direct Download**  
   Leave this option **disabled** (grey toggle).  
   > If enabled, CloudStack will automatically fetch the ISO from the URL.

5. **Zone**  
   Select the zone that was created earlier: `zone-20`

6. **Bootable**  
   Enable this option (blue toggle).  
   > This marks the ISO as bootable so it can be used to install VMs.

7. **OS Type**  
   Select the appropriate operating system:  `Other Ubuntu (64-bit)`

---

## Note:
All other fields not mentioned above should be **left with their default values** unless there are specific advanced configurations required.

