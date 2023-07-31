# DEVOPS SHORTCUTS FROM MY FIRST JOB (2020 VERSION)
- [Number 1: Change owner and group owners of a file in linux](#Number_1)
- [Number 2: Change file permission to read only for all the users for a file](#Number_2)
- [Number 3: Create a Linux User with non-interactive shell](#create-a-linux-user-with-non-interactive-shell)
- [Number 4 - Create a group](#create-a-group)
- [Number 5 - MariaDB Troubleshooting](#mariadb-troubleshooting)
- [Number 6 - Linux Archives](#linux-archives)
- [Number 7 - Linux SSH Authentication](#linux-ssh-authentication)
- [Number 8 - Linux Remote Copy](#linux-remote-copy)
- [Number 9 - Haproxy LBR Troubleshooting](#haproxy-lbr-troubleshooting)
- [Number 10 - Linux Run Levels](#linux-run-levels)
- [Number 11 - Create a user](#create-a-user)
- [Number 12 - Linux String Substitute](#linux-string-substitute)
- [Number 13 - Create a Cron Job](#create-a-cron-job)
- [Number 14 - Linux Banner](#linux-banner)
- [Number 15 - Linux Services](#linux-services)
- [Number 16 - Disable Root Login](#disable-root-login)
- [Number 17 - Linux User Without Home](#linux-user-without-home)
- [Number 18 - Linux User Expiry](#linux-user-expiry)
- [Number 19 - Apache Troubleshooting](#apache-troubleshooting)
- [Number 20 - Linux Firewalld Rules](#linux-firewalld-rules)
- [Number 21 - NFS Troubleshooting](#nfs-troubleshooting)
- [Number 22 - DNS Troubleshooting](#dns-troubleshooting)
- [Number 23 - Selinux Installation](#selinux-installation)

## Number_1 
### Change owner and group owners of a file in linux

```
chown root:root /etc/filename.conf
```

## Number_2 

### Change file permission to read only for all the users for a file

```
chmod 774 /etc/filename.conf
```
## Change file permission/ACL for a particular user to have zero permission for a file

```
setfacl -m u:userx:000 /tmp/filename
```
## Create a Linux User with non-interactive shell

Create a user with a non-interactive shell.

### Solution

```bash
# SSH into the app server
ssh steve@server02

# Create user
sudo useradd mark -s /sbin/nologin

# Check if default shell is correctly set
sudo cat /etc/passwd | grep mark

# Exit from app server
exit
```

## Create a group

There are specific access levels for users defined by devops team. Rather than providing access levels to every individual user, we decided to create groups with required access levels and add users to that groups as needed. See the following requirements:

a. Create a group named *new_group** in all App servers

b. Add the user **stark** to **verisk_streaming_noc** in all App servers. (create the user if not present already)

### Solution

> **Repeat steps in other app servers**

```bash
# SSH into the app server 1
ssh tony@application01

# Check if user exists
sudo id stark

sudo cat /etc/passwd | grep stark

# Add the group
groupadd verisk_streaming_noc

# Add the user (if it does not exists)
useradd -G verisk_streaming_noc stark

# Run Step 3 to check if the user created successfully and correct group is assigned

# Modify the user (optional only if the user existed before)
usermod -aG verisk_streaming_noc stark # Will change users primary group
usermod -g verisk_streaming_noc stark # Will add the user to a secondary group

# Exit from the app server
exit

# Execute Step 2 to check if the correct group is assigned
```

## MariaDB Troubleshooting

There is a critical issue going on with the **verisk_streaming** application in **Vancouver**. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.

Look into the issue and fix the same.

### Solution

```bash
# SSH into db server
ssh peter@stdb01

# Check status of mariadb service
sudo systemctl status mariadb

# If service is stopped, the try to start it
sudo systemctl start mariadb

# If there are errors in starting the service check mariadb logs
sudo journelctl -u mariadb.service # use -b switch to see logs from current the current boot

# From logs I found out that there was a permission issue. So started checking owners of the folders

# mysql
ll -lsd /var/lib/mysql/

# change owner if it is not set to mysql (and mysql group)
chown mysql:mysql /var/lib/mysql/

# mysqld
ll -lsd /var/run/mysqld/

# change owner if it is not set to mysql (and mysql group)
chown mysql:mysql /var/run/mysqld/

# mariadb
ll -lsd /var/run/mariadb/

# change owner if it is not set to mysql (and mysql group)
chown mysql:mysql /var/run/mariadb/

# Start the service again
sudo systemctl start mariadb

# Check service status
sudo systemctl status mariadb

# Exit from db server
exit
```

## Linux Archives

On **verisk_streaming** storage server in **Vancouver DC** there is a storage location **/data** which is used by different developers to keep their data (no confidential data). One of the developers **ravi** has raised a ticket and asked for a copy of his/her data present in **/data/ravi** directory on storage server. **/home** is an FTP location on storage server where developers can download their data. Below are the instructions shared by the system admin team to accomplish the task:

a. Make a **ravi.tar.gz** compressed archive of **/data/ravi** directory and move the archive to **/home** directory on Storage Server.

### Solution

```bash
# SSH to storage server 
ssh natasha@storage1server

# Check data folder and see if a folder called ravi exists
ls -l  /data/

# Archive and zip it.
sudo tar -cvzf ravi.tar.gz /data/ravi

# c - Creates a new .tar archive file
# v - Verbosely show the .tar file progress
# z - Create a compressed gzip archive file
# f – File name type of the archive file

# Move tar.gz file to the /home directory.
sudo mv ravi.tar.gz /home/

# Check if the file is moved successfully to /home directory.
ls -l /home/

# Exit from storage server
exit
```

## Linux SSH Authentication

The system admins team of **Verisk** has set up some scripts on **jump host** that run on regular intervals and perform operations on all app servers in **Vancouver Datacenter**. To make these scripts work properly we need to make sure **thor** user on jump host haspassword-less SSH access to all app servers through their respective sudo users. Based on the requirements, perform the following:

Set up a password-less authentication for user **thor** on jump host to all app servers through their respective sudo users.

### Solution

```bash
# Use ssh-keygen to generate a ssh key pair on jumhost. It will be saved in the .ssh directory.
ssh-keygen -t rsa -b 4096

# t - Public key algorithm to use. acceptable values are rsa, dsa, ecdsa, ed25519
# b - keysize
# Ref - https://www.ssh.com/academy/ssh/keygen#what-is-ssh-keygen?

# Check .ssh directory. You will find two files id_rsa and id_rsa.pub		
ll /home/thor/.ssh
		
# Copy public key to all the app servers. If you get key not found or ID not found then try ssh-copy-id -i /home/thor/.ssh/id_rsa.pub tony@application01
ssh-copy-id tony@application01
ssh-copy-id steve@application02
ssh-copy-id banner@application03

# Verify. You should be able to login without specifying password
ssh tony@application01
ssh steve@application02
ssh banner@application03

# Exit from app servers
exit
```

## Linux Remote Copy

One of the **verisk_streaming** developers has copied confidential data on the jump host in **Vancouver DC**. That data must be copied to one of the app servers. Because developers do not have access to app servers, they asked the system admins team to accomplish the task for them.

Copy **/tmp/verisk_streaming.txt.gpg** file from jump server to **App Server 3** at location **/home/code**.

### Solution

```bash
# Check existence of file in the specified directory.
ls -l /tmp
sudo cat /tmp/verisk_streaming.txt.gpg

# Copy the file from the jump server to any directory in the App Server 3 - using scp
# thor user doesn't have permissions to copy files to the /home directory of the App Server. 
# So we will first copy the file to the /temp directory on the App server.
sudo scp -r /tmp/verisk_streaming.txt.gpg banner@application03:/tmp

# SSH to the app server and move the file from /tmp directory to the required directory.
ssh banner@application03
sudo mv /tmp/verisk_streaming.txt.gpg /home/code

# Verify that file is moved successfully to the target directory.
ls -l /home/code/
sudo cat /tmp/verisk_streaming.txt.gpg

# Exit from the app server
exit
```

## Haproxy LBR Troubleshooting

**Verisk** has an application running on **verisk_streaming** infrastructure in **Vancouver Datacenter**. The monitoring tool recognised that there is an issue with the **haproxy** service on **LBR** server. That needs to fixed to make the application work properly.

Troubleshoot and fix the issue, and make sure **haproxy** service is running on **verisk_streaming** LBR server.

### Solution

```bash
# SSH to LBR Server
ssh loki@stlb01

# Check haproxy
sudo systemctl status haproxy

# Logs in the status reports an error in the configuration file : /etc/haproxy/haproxy.cfg. Missing timeouts for frontend main

# Check if the configuration is valid
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# This will more likely point you to the correct location where the error occured. In my case, there was a typo (clients instead of client)

# Open the configuration file in vi
sudo vi /etc/haproxy/haproxy.cfg

# Fix the type issue and save the file

# Check if the configuration is valid
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# If valid, the proceed otherwise continue checking and fixing config issue

#  Start the service
sudo systemctl start haproxy

# Check status
sudo systemctl status haproxy

# Enable service to start on boot
sudo systemctl enable haproxy
```

## Linux Run Levels

New tools have been installed on the app server in **Vancouver Datacenter**. Some of these tools can only be managed from the graphical user interface. Therefore, there are requirements for these app servers.

On all App servers in **Vancouver Datacenter** change the default runlevel so that they can boot in **GUI (graphical user interface)** by default.

### Solution

> **Repeat steps in other app servers**

```bash
# SSH into the app server
ssh tony@application01

# Check current run level
sudo systemctl get-default

# Change to graphical user interface
sudo systemctl set-default graphical.target

# Check status
systemctl status graphical.target

# Start if stopped
systemctl start graphical.target

# Enable to start at boot
systemctl enable graphical.target
```

## Create a user

For some security reasons **Verisk** security team has decided to use custom Apache users for each web application hosted there rather than its default user. Since this is going to be the Apache user so it should not use the default home directory. Create the user as per requirements given below:

a. Create a user named **anita** on the **App server 1** in Vancouver Datacenter.

b. Set UID to **1619** and its home directory to **/var/www/anita**

### Solution

```bash
#  SSH to App server
ssh tony@application01

# Add the user.
sudo useradd -m -d /var/www/anita anita -u 1619

# m - create home directory if it does not exist
# d - user will be created in the specified home directory
# u - assigns specified user ID

# verify details for the user
sudo id anita
sudo /etc/passwd | grep anita
```

## Linux String Substitute

Replace all occurrences of the string **something** to **execute** on the XML file **/root/some.xml** located in the backup server.

### Solution

```bash
# SSH to the server
ssh clint@someserver

# Check About text in the XML file
sudo cat /root/some.xml | grep About

# Check the occurrence  of About text in the XML file as there are few in a different case
sudo cat /root/some.xml | grep About | wc -l

# Replace
sudo sed -i 's/something/execute/g' /etc/some.xml

# i - By default sed is case-sensitive, specify -i for case-insensitive search
# s/ - search
# /g - all occurrences

# Check again if the replacement was successful
sudo cat /root/verisk_streaming.xml | grep Cloud | WC -l
```

## Create a Cron Job

The **verisk_streaming** system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in **Vancouver DC** on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:

a. Install **cronie** package on all **verisk_streaming** app servers and start **crond** service.

b. Add a cron ***/5 * * * * echo hello > /tmp/cron_text** for **root** user.

### Solution

> **Do this for all app servers**

```bash
# SSH into app server 1
ssh tony@application01

# Switch to root user
# sudo su -

# Install cronie
yum install -y cronie

# Start crond service
systemctl start crond

# Check crond service
systemctl status crond

# This will allow us to add a cron job
crontab -e

# then add this to the file
*/5 * * * * echo hello > /tmp/cron_text

# to verify if cronjob is created successfully
crontab -l -u root

# watch the folder to check if our cronjob is working as expected
watch -n 5 ls -l /tmp

# After 5 seconds, we will see a new file called cron_text

# Check if a cron job is created for the root user in cron directory
ls /var/spool/cron
cat /var/spool/cron/root
```

## Linux Banner

During the monthly compliance meeting, it was pointed out that several servers in the **Vancouver DC** do not have a valid banner. The security team has provided serveral approved templates which should be applied to the servers to maintain compliance. These will be displayed to the user upon a successful login.

Update the **message of the day** on all application and db servers for **verisk_streaming**. Make use of the approved template located at **/root/verisk_streaming_banner** on jump host

### Solution

> **Do this on all app servers**

```bash
# Check banner file defails on host
ll -lsd /tmp/verisk_streaming_banner

# Copy banner to the /tmp directory on app server 1
sudo scp -r /root/verisk_streaming_banner tony@application01:/tmp

# Run move command on the app server 1
ssh -t tony@application01 'sudo mv /tmp/verisk_streaming_banner /etc/motd'

# Validate
ssh tony@application01

# You should see the banner now

# If incase, scp fails, the most proable reason is that ssh client is not installed
sudo yum install openssh-clients
```

## Linux Services

As per details shared by the development team, the new application release has some dependencies on the back end. There are some packages/services that need to be installed on all app servers under **Vancouver Datacenter**. As per requirements please perform the following steps:

a. Install **httpd** package on all the application servers.

b. Once installed, make sure it is enabled to start during boot.

### Solution

> **Do this on all app servers**

```bash
# SSH into app server 1
ssh tony@application01

# Switch user to root
sudo su -

# install httpd
yum install httpd

# enable to start during boot
systemctl enable httpd

# start the service
systemctl start httpd

# Check status
systemctl status httpd

# optional but also useful
systemctl list-unit-files | grep httpd

# Exit the server
exit
```

## Disable Root Login

After doing some security audits of servers, **Verisk** security team has implemented some new security policies. One of them is to disable direct root login through SSH.

Disable direct SSH root login on all app servers in **Vancouver Datacenter**.

### Solution

> **Perform these steps on all app servers**

```bash
# SSH to app server 1
ssh tony@application01

# Edit the /etc/ssh/sshd_config and change change "PermitRootLogin yes" to "PermitRootLogin no", Uncomment if commented
sudo vi /etc/ssh/sshd_config

# Check if the changes are applied
sudo cat /etc/ssh/sshd_config | grep PermitRoot

# Reload configuration
sudo systemctl reload sshd

# Check service status
sudo systemctl status sshd

# Exit the app server
exit
```

## Linux User Without Home

The system admins team of **Verisk** has set up a new tool on all app servers, as they have a requirement to create a service user account that will be used by that tool. They are finished with all apps except for **App 1** in **Vancouver Datacenter**.

Create a user named **kareem** in **App Server 1** without a home directory.

### Solution

```bash
# SSH into App server 1
ssh tony@application01

# Create user without a home directory
sudo useradd -M kareem

# Verify that home directory is not created
ls -l /home

# Exit the app server
exit
```

## Linux User Expiry

A developer **John** has been assigned **verisk_streaming** project temporarily as a backup resource. As a temporary resource for this project, we need a temporary user for **John**. It’s a good idea to create a user with a set expiration date so that the user will not be able to access servers beyond that point.

Therefore, create a user named **john** on the **App Server 1**. Set **expiry date** to **2021-12-07** in **Vancouver Datacenter**. Make sure the user is created as per standard and is in lowercase.

### Solution

```bash
# SSH to App server 1
ssh tony@application01

# Add user with expiry
sudo useradd -e 2021-12-07 john 

# If the user already exists then use the following command
sudo chage -E 2021-12-07 john 

# verify
sudo chage -l john 

# Exit the app server
exit
```

## Apache Troubleshooting

**Verisk** utilizes monitoring tools to check the status of every service, application, etc. running on the systems. The monitoring system identified that Apache service is not running on some of the **verisk_streaming Application** Servers in **Vancouver Datacenter**.

Identify the faulty **verisk_streaming Application Servers** and fix the issue. Also, make sure Apache service is up and running on all **verisk_streaming Application Servers**. Do not try to stop any kind of firewall that is already running.

Apache is running on **5001** port on all **verisk_streaming Application Servers** and its document root must be **/var/www/html** on all app servers.
  
Finally you can test from **jump host** using curl command to access Apache on all app servers and it should work fine. E.g. **curl http://172.16.238.10:5001/**

### Solution

> Perform the steps on all app servers

```bash
# SSH into App Server 1
ssh tony@application01

# Switch to root user
sudo su -

# Check httpd service status
systemctl status httpd

# If it is stopped, then try to start it
systemctl start httpd

# If unable to start check logs
systemctl status httpd

# There was a syntax error on a particular line in the /etc/httpd/conf/httpd.conf

# Open the conf file and go to the error line
vi /etc/httpd/conf/httpd.conf

# Listen was enclosed in double quotes. Remove the double quotes and also make sure port number is correct
# "Listen 5001" should be Listen 5001
# Save conf and reload service
# Status shows another error in the conf file
# Open it again
# This time the error was a semi-colon at the end of DocumentRoot /var/www/html. Fix this. Also make sure DocumentRoot is pointing to correct directory
# Tried starting the service again
# Another error in the config
# This time ServerRoot value was incorrect. there was a semi-colon at the end of "/etc/httpd"
# Fixed that and started the service again.
# Success this time

# Exit from the app server
exit

# curl from the jump host
curl http://172.16.238.10:5001/

# Success
```

## Linux Firewalld Rules

The **verisk_streaming** system admins team recently deployed a web UI application for their backup utility running on the **verisk_streaming backup server** in **Vancouver Datacenter**. The application is running on port **6200**. They have **firewalld** installed on that server. The requirements that have come up include the following:

Open all incoming connection on **6200/tcp** port. Zone should be **public**.

### Solution

```bash
# Connect via SSH to the Backup server
ssh clint@stbkp01

# Check status of firewalld
sudo systemctl status firewalld

# Lists config for firewall
sudo firewall-cmd --list-all --zone=public							

# Allowing all incoming traffic on port 6200 with zone public
sudo firewall-cmd --zone=public --permanent --add-port=3000/tcp

# Reload firewalld
sudo firewall-cmd --reload

# Restart firewalld
sudo systemctl restart firewalld

# Verify
sudo firewall-cmd --zone=public --list-all

# Check status of firewalld
sudo systemctl status firewalld

# Exit from server
exit
```

## NFS Troubleshooting

The **verisk_streaming** production support team was trying to fix issues with their storage server. The storage server has a shared directory **/opt**, which is mounted on all app servers at location **/var/www/html** so that whatever data they store on storage server under **/opt** can be shared among all app servers. Somehow NFS server is broken and having some issues.

Identify the root cause of the issue and fix it to make sure sharing works fine among all app servers and storage server.

### Solution

```bash
# SSH to the Storage Server.
ssh natasha@storage1server

# Switch to root user
sudo su -

# Check status of nfs server and rpcbind (This utility maps RPC services to the ports on which they listen)
systemctl status nfs-server && systemctl status rpcbind

# They are stopped right now. Stop them if they are started

# Checks contents of export file and edit it. This should contain the ip address of the three App servers.
cat /etc/exports

vi /etc/exports

# Mounting/shared directory has to match with the one given in the problem statement 
# /opt 172.16.238.10(rw,sync,no_subtree_check,no_root_squash,fsid=0)
# /opt 172.16.238.11(rw,sync,no_subtree_check,no_root_squash,fsid=0)
# /opt 172.16.238.12(rw,sync,no_subtree_check,no_root_squash,fsid=0)

# Start nfs-server and rpcbind services
systemctl start nfs-server && systemctl start rpcbind

# To ensure that file system shared directory will reflect on the Appservers, make sure to export file system (exportfs)
exportfs -r

# Print and check the list of exported filesystems.
showmount -e storage1server

# Connect via SSH to each App server. For each one, do the next steps.

ssh tony@application01    # App server 1
ssh steve@application02   # App server 2
ssh banner@application03  # App server 3

# Start nfs-server and rpcbind services
systemctl start nfs-server && systemctl start rpcbind

# Ensure App server sees the filesystem and the correct shared shared directory(showmount).
showmount -e storage1server

# Mount the directory into the App server
mount -t nfs storage1server:/opt /var/www/html

# Verify directory is mounted.
df -h

# Or

mount | grep nfs

# Create a file inside /var/www/html - this should be seen by all App servers and storage server.
sudo touch /opt/test

# On Storage server, check the shared directory
ll /opt

# Exit from all servers
exit
```

## DNS Troubleshooting

The system admins team of Verisk has noticed intermittent issues with DNS resolution in several apps . **App Server 3** in **Vancouver Datacenter** is having some DNS resolution issues, so we want to add some additional DNS nameservers on this server.

As a temporary fix we have decided to go with Google public DNS. Please make appropriate changes on this server.

### Solution

```bash
# SSH to app server 3.
ssh banner@application03

# Verify if DNS is really not working by doing a ping test.
ping www.google.com

# Edit /etc/resolv.conf and add nameserver for Google
sudo vi /etc/resolv.conf
# add nameserver 8.8.8.8

# Test connectivity
ping www.google.com

#   Exit from server
exit
```

## Selinux Installation

The Verisk security team recently did a security audit of their infrastructure and came up with ideas to improve the application and server security. They decided to use SElinux for an additional security layer. They are still planning how they will implement it; however, they have decided to start testing with app servers, so based on the recommendations they have the following requirements:

Install the required packages of SElinux on **App server 3** in **Vancouver Datacenter** and disable it permanently for now; it will be enabled after making some required configuration changes on this host. Don't worry about rebooting the server as there is already a reboot scheduled for tonight's maintenance window. Also ignore the status of SElinux command line right now; the final status after reboot should be **disabled**.

### Solution

```bash
# SSH to the App Server 3
ssh banner@application03

# Install Selinux
sudo yum install selinux*

# This tool is used to get the status of a system running SELinux.
sudo sestatus

# Edit the /etc/selinux/config file as ROOT then verify
sudo vi /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#   enforcing - SELinux security policy is enforced.
#   permissive - SELinux prints warnings instead of enforcing.
#   disabled - No SELinux policy is loaded.
SELINUX=enforcing	                                # Change this to disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

# Exit from the app server
exit
```
