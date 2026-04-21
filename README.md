# Multi-Tier Web Application Deployment (Libvirt + Virsh)
 
## Overview

This project demonstrates a **multi-tier web application deployment** using **KVM-based virtualization (libvirt + virsh)** with **automated shell-script provisioning**.

VM infrastructure is created using `virsh`, and services are configured using **predefined automation scripts** for reproducible setup.
---
 
## Architecture

| VM Name | Role               | OS           | IP Address      | Main Services                  |
|---------|--------------------|--------------|-----------------|-------------------------------|
| db01    | Database Server    | CentOS Stream 9 | 192.168.56.15  | MariaDB                       |
| mc01    | Cache Server       | CentOS Stream 9 | 192.168.56.14  | Memcached                    |
| rmq01   | Message Broker     | CentOS Stream 9 | 192.168.56.13  | RabbitMQ                     |
| app01   | Application Server | CentOS Stream 9 | 192.168.56.12  | Tomcat (Java 17), Maven, Git |
| web01   | Web Server / Proxy | Ubuntu Jammy  | 192.168.56.11  | Nginx (Reverse Proxy)        |

---

## Features

- Automated provisioning of base packages and services via KVM-based Libvirt + virsh tool with shell scripts
- Proper firewall configuration on each VM
- Private networking with fixed IPs for inter-VM communication
- Modular architecture simulating a production-grade environment
- Manual provisioning scripts for hands-on learning
- Planned automation using Ansible in future stages

---

### Request Flow
 
```
Client → Nginx → Tomcat → (RabbitMQ + Memcached + MySQL)
```
 
---
 
## Technologies Used
 
- **Hypervisor:** KVM (libvirt, virsh)
- **OS:** CentOS Stream 9, Ubuntu 22.04
- **Web Server:** Nginx
- **Application Server:** Apache Tomcat (Java 17)
- **Database:** MariaDB (MySQL)
- **Cache:** Memcached
- **Message Broker:** RabbitMQ
- **Build Tool:** Maven
---

## Setup Instructions

### 1. Clone this repository 
```bash
git clone [https://github.com/Shihab369/vprofile-infra.git](https://github.com/Shihab369/vprofile-infra.git)
cd vprofile-infra
```

--- 
## VM Creation (Libvirt / Virsh)
 
### 2. Create Virtual Machines
 
Example command used for VM creation:
 
```bash
virt-install \
--name db01 \
--memory 1024 \
--vcpus 1 \
--disk size=10 \
--os-variant centos-stream9 \
--network network=default \
--graphics none \
--console pty,target_type=serial \
--location 'http://mirror.centos.org/centos/9-stream/BaseOS/x86_64/os/' \
--extra-args 'console=ttyS0,115200n8 serial'
```
 
Repeat similar commands for:
- mc01
- rmq01
- app01
- web01
### 3. Check VM Status
 
```bash
virsh list --all
```
 
### 4. Start / Stop VMs
 
```bash
virsh start db01
virsh shutdown db01
```
 
### 5. Access VM Console
 
```bash
virsh console db01
```
 
---
 
## Provisioning
 
All services were configured manually inside each VM.
 
Detailed step-by-step commands are available in: `scripts/commands.md`
 
---
 
## Services Setup Summary
 
### 1. MySQL (db01)
 
- Installed MariaDB
- Secured installation
- Created database: `accounts`
- Created user: `admin`
- Imported initial schema
### 2. Memcached (mc01)
 
- Installed memcached
- Configured to listen on all interfaces
- Opened required firewall ports
### 3. RabbitMQ (rmq01)
 
- Installed RabbitMQ server
- Created user `test`
- Assigned admin role
- Configured permissions
### 4. Tomcat (app01)
 
- Installed Java 17
- Installed Apache Tomcat
- Built application using Maven
- Deployed WAR file to Tomcat
### 5. Nginx (web01)
 
- Installed Nginx
- Configured reverse proxy to `app01:8080`
- Enabled site configuration
---
 
## Directory Structure
 
```
vprofile-infra/
├── scripts
│   ├── commands.md
│   ├── memcache.sh
│   ├── mysql.sh
│   ├── nginx.sh
│   ├── rabbitmq.sh
│   └── tomcat.sh
├── Vagrantfile
└── README.md
```
 
---
 
## Troubleshooting

- Ensure KVM/libvirt service is running before starting any VM:
  ```bash
  sudo systemctl status libvirtd
  ```
- Check service status inside VM:
  ```bash
  systemctl status <service-name>
  ```
- Restart a failed service:
  ```bash
  firewall-cmd --list-all
  ```
- Verify firewall rules (if ports not accessible):
  ```bash
  firewall-cmd --list-all
  ```  
---
 
## Future Improvements
 
- Automate provisioning using Ansible
- Add monitoring (Prometheus, Grafana)
- Containerize services using Docker
- Deploy using Kubernetes
---
 
## Getting Started
 
1. Ensure libvirt and virsh are installed on your KVM host
2. Create VMs using the virt-install commands provided above
3. Follow the provisioning steps in `scripts/commands.md` for each VM
4. Access the application at `http://192.168.56.11` (Nginx web01)
---
