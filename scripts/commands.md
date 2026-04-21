## Overview

This setup focuses on **infrastructure provisioning, service configuration, and inter-service communication** without using full automation tools.
 
---
 
## Services Used
 
1. Nginx → Web Service (Reverse Proxy)
2. Tomcat → Application Server
3. RabbitMQ → Message Broker / Queue Service
4. Memcached → Caching Layer
5. MySQL → Database Service
 
---
 
## Architecture
 
| VM Name | Role              | Service         | IP Address     |
|----------|------------------|----------------|----------------|
| db01     | Database Server   | MySQL          | 192.168.56.15  |
| mc01     | Cache Server      | Memcached      | 192.168.56.14  |
| rmq01    | Message Broker    | RabbitMQ       | 192.168.56.13  |
| app01    | Application Node  | Tomcat + Maven | 192.168.56.12  |
| web01    | Web Proxy         | Nginx          | 192.168.56.11  |
 
---
 
## Prerequisites
 
- Linux host with libvirt + virsh installed
- Multiple VMs running and reachable via SSH
- Private network configured
- `/etc/hosts` configured for hostname resolution
 
---
 
## Common Setup (All VMs)
 
### 1. Access VM
 
Connect to VM using virsh or SSH.
 
```bash
virsh console <vm-name>
```
 
### 2. Verify Host Configuration
 
Ensures inter-VM communication works.
 
```bash
cat /etc/hosts
```
 
### 3. Update System Packages
 
```bash
dnf update -y
```
 
---
 
## Service Configuration
 
### 1. MySQL (db01)
 
#### Install MariaDB
 
```bash
dnf install epel-release -y
dnf install mariadb-server git -y
```
 
#### Start Service
 
```bash
systemctl start mariadb
systemctl enable mariadb
```
 
#### Secure Installation
 
```bash
mysql_secure_installation
```
 
#### Create Database and User
 
```bash
mysql -u root -p
create database accounts;
grant all privileges on accounts.* to 'admin'@'%' identified by 'admin123';
flush privileges;
exit;
```
 
#### Import Database
 
```bash
git clone -b local https://github.com/hkhcoder/vprofile-project.git /tmp/app
mysql -u root -p accounts < /tmp/app/src/main/resources/db_backup.sql
```
 
#### Open Firewall
 
```bash
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```
 
---
 
### 2. Memcached (mc01)
 
#### Install Memcached
 
```bash
dnf install memcached -y
```
 
#### Start Service
 
```bash
systemctl start memcached
systemctl enable memcached
```
 
#### Allow Remote Access
 
```bash
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
systemctl restart memcached
```
 
#### Open Firewall
 
```bash
firewall-cmd --add-port=11211/tcp --permanent
firewall-cmd --reload
```
 
---
 
### 3. RabbitMQ (rmq01)
 
#### Install RabbitMQ
 
```bash
dnf install -y centos-release-rabbitmq-38
dnf install -y rabbitmq-server
```
 
#### Start Service
 
```bash
systemctl enable --now rabbitmq-server
```
 
#### Create User
 
```bash
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```
 
#### Enable Remote Access
 
```bash
echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config
systemctl restart rabbitmq-server
```
 
#### Open Firewall
 
```bash
firewall-cmd --add-port=5672/tcp --permanent
firewall-cmd --reload
```
 
---
 
### 4. Tomcat (app01)
 
#### Install Java
 
```bash
dnf install java-17-openjdk git wget -y
```
 
#### Install Tomcat
 
```bash
cd /tmp
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
tar xzvf apache-tomcat-10.1.26.tar.gz
mv apache-tomcat-10.1.26 /usr/local/tomcat
```
 
#### Create User
 
```bash
useradd --shell /sbin/nologin tomcat
chown -R tomcat:tomcat /usr/local/tomcat
```
 
#### Build Application
 
```bash
git clone -b local https://github.com/hkhcoder/vprofile-project.git
cd vprofile-project
mvn install
```
 
#### Deploy Application
 
```bash
cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
systemctl restart tomcat
```
 
#### Open Firewall
 
```bash
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```
 
---
 
### 5. Nginx (web01)
 
#### Install Nginx
 
```bash
apt update
apt install nginx -y
```
 
#### Configure Reverse Proxy
 
```bash
cat <<EOT > /etc/nginx/sites-available/vproapp
upstream app {
    server app01:8080;
}
server {
    listen 80;
    location / {
        proxy_pass http://app;
    }
}
EOT
```
 
#### Enable Site
 
```bash
ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
```
 
#### Restart Service
 
```bash
systemctl restart nginx
```
 
---
 
## Execution Order
 
**db01 → mc01 → rmq01 → app01 → web01**
 
### Reason:
 
- DB must be ready first
- Cache and MQ support backend services
- App depends on all backend services
- Nginx is entry point
 
---
 
## Access Application
 
```
http://192.168.56.11
```
 
---
