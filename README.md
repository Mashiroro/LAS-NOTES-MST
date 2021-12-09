# LAS-NOTES-MST
LAS MST COMMANDS AND NOTES


# Table of contents

- [Commands](#commands)
- [User Management](#user-management)
  - [Users & Groups](#users--groups)
  - [Disable login for account](#disable-login-for-account)
  - [Sudo (/etc/sudoers)](#sudo-etcsudoers)
- [File Permissions](#file-permissions)
  - [File ACL](#file-acl)
- [Networking](#networking)
  - [Configure](#configure)
  - [Show networking information](#show-networking-information)
- [Security](#security)
  - [SELinux (Policy)](#selinux-policy)
  - [SELinux Context](#selinux-context)
  - [Allow anonymous upload for FTP](#allow-anonymous-upload-for-ftp)
- [Packages](#packages)
  - [Installer](#installer)
  - [Usage](#usage)
- [Services & applications](#services--applications)
  - [Usage](#usage)
  - [Services](#services)
- [FTP (/etc/vsftpd/vsftpd.conf)](#ftp-etcvsftpdvsftpdconf)
  - [Useful commands](#useful-commands)
  - [Anonymous](#anonymous)
  - [FTP welcome banner](#ftp-welcome-banner)
  - [Enable FTP service on boot](#enable-ftp-service-on-boot)
  - [Allow local users to login](#allow-local-users-to-login)
  - [Enable write command](#enable-write-command)
  - [Add FTP permanently to the firewall](#add-ftp-permanently-to-the-firewall)
  - [Chroot (vsftpd.conf)](#chroot-vsftpdconf)
- [SSH (/etc/ssh/sshd_config)](#ssh-etcsshsshd_config)
  - [Change listening port](#change-listening-port)
  - [Add SSHD to Firewall](#add-sshd-to-firewall)
  - [Add SSHD to on boot](#add-sshd-to-on-boot)
- [NFS](#nfs)
  - [Export the NFS](#export-the-nfs)
  - [Firewall](#firewall)
  - [Mounting](#mounting)
  - [Unmount the NFS](#unmount-the-nfs)
- [Apache (/etc/httpd/conf/httpd.conf)](#apache-etchttpdconfhttpdconf)
  - [Accessing](#accessing)
  - [Firewall](#firewall)
  - [/etc/httpd/conf/httpd.conf](#etchttpdconfhttpdconf)
  - [Server Status](#server-status)
  - [Change performance](#change-performance)
  - [Apache Logs](#apache-logs)
  - [Basic Access Control](#basic-access-control)
- [Vhost](#vhost)
  - [Configure VHost Client & Server (/etc/hosts)](#configure-vhost-client--server-etchosts)
  - [Create files for Vhost Webpage](#create-files-for-vhost-webpage)
  - [Access Control](#access-control)
  - [Vhost Apache (Put in /etc/httpd/conf.d/\<name>.conf or default area)](#vhost-apache-put-in-etchttpdconfdnameconf-or-default-area)
  - [Configure Access control to enable co-authoring or web content](#configure-access-control-to-enable-co-authoring-or-web-content)
- [Squid (/etc/squid/squid.conf)](#squid-etcsquidsquidconf)
  - [Logs](#logs)
- [Firewall](#firewall)


# Commands
```
# host name and info
hostname
hostnamectl

# create file
touch <file name>

# read from the last part of the file
tail <number> <filename>

# create a directory
mkdir <folder name>

# remove
rm <file>
rm -r

# using grep to find the passwd file for any user id called "NAME"
cat /etc/passwd | grep -o "^[^:]*" | grep NAME

# using AWK to find any userid called "NAME"
awk -F ':' '$1 ~/NAME/ {print $1}' /etc/passwd
awk -F ':' '{print $1}' /etc/passwd | grep ^NAME
```

# User Management
## Users & Groups
```
# change password
passwd

# create new account
useradd <username>

# add a user to a group
groupadd <username>

# display information of the user
id <username>

# display the group of the user
groups <username>

# change password for the use
passwd <username>

# change the group of the user
usermod –aG <groupname> <username>
chgrp <groupname> <username>

# login as user
su <username>

# login as root with sudo
sudo su
```
## Disable login for account
```
nano /etc/passwd

# change the back section from /bin/bash to:
/sbin/nologin
```
## Sudo (/etc/sudoers)
```
# Give all sudo perms
user ALL=ALL

# Disable root login
nano /etc/passwd
root:x:0:0:root:/root:/sbin/nologin	
```
# File Permissions
>7 = RWX
>6 = RW
>5 = RX
>4 = R
>3 = WX
>2 = W
>1 = X
>t = sticky bit (only allows the owner to delete the file)
>s = run the file as the owner. Also creates automatically changes the grp or owner of the file depending on configuration.
>u = owner
>g = group
>o = other

```
chmod <perm> <filename>
chown <user> <filename>
ls -la <filename>

# RWX to group
chmod g+=7 test.txt

# full access
chmod 7(owner)7(group)7(others) test.txt
chmod 777 test.txt

# removes others permissions
chmod o= <file>
```
## File ACL
```
# get file ACL
getfacl <folder>

# set the file ACL
setfacl -m <PERMISSION>

permission:
U:<USERNAME>:<PERMISSION>
G:<GROUP NAME>:<PERMISSION>
O:<USERNAME>:<PERMISSION>

example:
U:MARRY:RWX
G:PPM:RW
```
# Networking
## Configure
```
# using CLI 
nmcli

# set the ip address, gateway and dns
nmcli connection modify <interface> ipv4.addresses <ip>/<CIDR> ipv4.gateway <ip> ipv4.dns <IP>

# change the modes
nmcli connection modify <interface> ipv4.method manual
nmcli connection modify <interface> ipv4.method auto

# delete the connection
# take note if you switch from manual to auto, the device will have 2 ip address
nmcli connection delete <interface>
nmcli device connect <interface>

# GUI editor
nmtui 

# restart the networking service
nmcli networking off  
nmcli networking on   
nmcli device disconnect <interface>
nmcli device connect <interface>
```

## Show networking information
```
# displayip paddress information
ip a
nmcli d show <interface card>

# get running services's port number
netstat -tunap | grep <service>

netstat -tunap | grep sshd
netstat -tunap | grep squid

# display the DNS
cat /etc/resolv.conf

route -n
netstat -apetul
netstat -plant
ss -l
ss -ta
ss -tp
```

# Security
## SELinux (Policy)
>Enforcing - action is prohibited and logged
>Permissive - action logged
>Disabled
```
# Get status of selinux
getenforce

# Change modes for se linux
/etc/sysconfig/selinux -> SELINUX=disabled
setenforce 0 #permissive
setenforce 1 #enforcing

# View first 15 SElinux booleans
getsebool -a | head -15

# GUI SElinux
dnf install policycoreutils-gui
```
## SELinux Context
```
ls -Z <file>
# Display SElinux credentials
ls -lZ

# Change context of file
chcon -t shadow_t <file>
chcon -t public_content <file>

# restore original context
restorecon <file>
```
## Allow anonymous upload for FTP
```
mkdir /var/ftp/<folder name>
chgrp ftp /var/ftp/<folder name>
chmod 730 /var/ftp/<folder name>

nano /etc/vsftpd/vsftpd.conf
anon_upload_enable=YES
write_enable=YES


setsebool -P allow_ftpd_anon_write True
setsebool -P allow_ftpd_full_access True
# or
setsebool -P allow_ftpd_anon_write=1
setsebool -P allow_ftpd_full_access=1
```
> group must have WRITE ACCESS
> group must be FTP

# Packages
## Installer
```
yum
dnf
rpm #installer for .rpm
```
## Usage
```
# install
dnf install <item>
yum install <item>

# search for package
dnf search <item>

# list installed packages
dnf list installed
dnf installed | grep <file>

# delete a package
dnf remove

# upgrade the package
dnf upgrade

# list all available repositories
dnf repolist all 

# list all enabled repositories
dnf repolist enabled

# see history of installed/deleted packages
dnf history
```

# Services & applications
## Usage
```
<service name> - ssh, ftp
<action> - restart, start, stop, enable

service <service name> <action>
systemctl <action> <service name>

# display services
systemctl list-unit-files --type service

# auto start at boot
systemctl enable <service>
```
## Services
```
sshd
firewalld
nmap

# Gui for Firewall
firewall-config
# GUI for SELinux
policycoreutils-gui 

vsftpd
openssh-server
nfs-server
nfs-utils
cockpit

# Apache
httpd
mod_ssl
squid
```

# FTP (/etc/vsftpd/vsftpd.conf)
> Default folder path: /var/ftp/pub
> ftpd_banner //Enable to prevent Information Disclosure!
> anonymous_enable //Disable!
> anon_upload_enable //Disable!
> local_enable
> write_enable
## Useful commands
```
dnf install vsftpd
dnf info vsftpd
service vsftpd start
service vsftpd stop
service vsftpd status
```
## Anonymous 
```
ftp <ip>
username: anonymous

# enable anonymous login
anonymous_enable=YES

# allow anonymous to upload files 
anon_upload_enable=YES
```
> for anonymous file upload check SElinux category.
## FTP welcome banner
```
ftpd_banner=<TEXT>
banner_file=<FOLDER PATH>
```

## Enable FTP service on boot
```
systemctl enable vsftpd
systemctl status vsftpd
```
## Allow local users to login
```
local_enable=YES
ftp <local user>@<ip>
```
## Enable write command
```
write_enable=YES
```


## Add FTP permanently to the firewall
```
firewall-cmd --permanent --zone=public --add-service=ftp
firewall-cmd --reload
firewall-cmd --list-all
```

## Chroot (vsftpd.conf)
>chroot_local_user //Enable chroot
>chroot_list_enable //Exempted from jail
>chroot_list_file //list of users to exempt chroot
>allow_writeable_chroot // allows chroot user to write in their home directory
>passwd_chroot_enable // jail the account in the FTP to the /home directory based on the /etc/passwd. Unable to go to any other directories. 
```
# Enable chroot for local users
chroot_local_user=YES 

# exempt a list of users from chroot
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list

# enable write 
allow_writeable_chroot=YES

# jail the account to it's /home directory
passwd_chroot_enable=YES
```
```
If:
chroot_local_user=NO
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
# User in chroot_list will be chroot jailed
```
# SSH (/etc/ssh/sshd_config)
usage
```
ssh <username>@<ip>
```
## Change listening port
```
Port 8222
```
Add this to sshd_config
```
#SELinux is preventing sshd from name_bind access on the tcp_socket port 8222
semanage port -a -t ssh_port_t -p tcp 8222

#check if SSHD has been added to se linux
semanage port -l | grep ssh
```
SElinux prevents SSHD to bind to other port numbers, hence enter this command to add it
```
service sshd restart
```
## Add SSHD to Firewall
```
firewall-cmd --zone=public --permanent --add-port=8222/tcp
firewall-cmd --reload
firewall-cmd --list-all
```
## Add SSHD to on boot
```
systemctl enable sshd 
service sshd status
```
There must be a "enabled" for SSHD for it to start on boot
# NFS
## Export the NFS
> folder must be created beforehand and have the permission RWX.
```
service nfs-server start

nano /etc/exports
<PATH TO FOLDER>    <IP ADDRESS>(<PERMISSION>,sync)

# re-export entries. Run this after editing the /etc/exports
exportfs -r

# check the exports
exportfs -v

# example of /etc/exports
/exports/data 192.168.30.128(rw,sync,root_squash) (ro,sync)
/exports/data 192.168.30.128(rw,sync,root_squash) 192.168.30.128(ro,sync)
```
> ro = read-only
> rw = read-write
> rwx = read-write-execute
> root_squash = client's root account are mapped to nodbody. (enabled by default) 
> no_root_squash = client root == server root 
> sync = synchronize all files 
> empty options e.g., (ro,sync) == *(ro,sync) 

## Firewall 
```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --reload
firewall-cmd --list-all
```
> NFS uses port 2049
## Mounting 
```
# prep
dnf list nfs-utils
dnf install nfs-utils
mkdir /mount/data 

# mount to the folder created earlier
mount <SERVER IP>:/exports/data /mount/data

# Get current information of the nfs
mount | grep nfs4
```
## Unmount the NFS
```
# umount
umount /mount/data

# force unmount (NOT RECOMMENDED)
umount -f /mount/data

# if its empty then its unmounted.
mount | grep nfs4
```
> if device is busy means a folder is opened either in GUI or terminal. 
# Apache (/etc/httpd/conf/httpd.conf)
## Accessing 
```
localhost
localhost/<webpage>
```

## Firewall
```
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --reload
firewall-cmd --list-all
```
## /etc/httpd/conf/httpd.conf
```
# set the folder where the website will run
DocumentRoot "/var/www/html"

# set the main webpage
DirectoryIndex index.html
```
## Server Status
> Show server-status (Recommended for localhost only)
> Change localhost to ip address or all accordingly
```
# configure server status
nano /etc/httpd/conf/httpd.conf

<Location /server-status>
    SetHandler server-status
    Order allow,deny
    Allow from localhost
</Location>

service httpd restart

# accessing
http://localhost/server-status
```
## Change performance 
```
nano /etc/httpd/conf/httpd.conf

StartServers 5 # Nodes
ThreadsPerChild 15 # Threads
ServerLimit 10
```
## Apache Logs
```
tail /var/log/httpd/access_log
tail /var/log/httpd/error_log
```
## Basic Access Control
>Set directory as page (Put in /etc/httpd/conf.d/\<name>.conf or default area)
```
<Directory /var/www/html/directory>
    Options -Indexes
</Directory>
```
### Prevent Directory Query
```
<Directory /var/www/html/directory>
    Options -Indexes
    Require all denied
    Require ip <ip address>
    Require local
</Directory>
```
# Vhost
## Configure VHost Client & Server (/etc/hosts)
```
<IP ADDRESS> <DOMAIN NAME> <NAME>
192.168.30.88 www.flowers.com flowers

10.10.10.10 www.domain.com admin.domain.com domain.com
```
## Create files for Vhost Webpage
```
mkdir /var/www/<folder name>
nano index.html
```
## Access Control
```
<Directory /var/www/FOLDERNAME>
 Options -Indexes
 Require all denied
 Require ip <IP ADDRESS>
 Require local
</Directory>
```
> Forbid access to all files, folders and subfiles for that specific ip address
> Require IP address give access to the specific IP address
> Require Local give access to the localhost

## Vhost Apache (Put in /etc/httpd/conf.d/\<name>.conf or default area)
```
nano /etc/httpd/conf.d/\<name>.conf
# or
nano /etc/httpd/conf/httpd.conf

<VirtualHost <YOUR IP ADDRESS>:80>
 ServerName www.domain.com
 DocumentRoot /var/www/<folder name>
 ErrorLog /var/log/httpd/<name>-error_log
 CustomLog /var/log/httpd/<name>-access_log combined
</VirtualHost>

systemctl httpd restart
```
> servername == /etc/hosts (2nd column)
> DocumentRoot == folder of the Vhost Webpage /var/www/FOLDERNAME
> Errorlog and CustomLog name should be the same

## Configure Access control to enable co-authoring or web content
```
# This will create any sub files/folder as the grp
chmod g+s /var/www/<folder>
```
# Squid (/etc/squid/squid.conf)
## Logs
```
grep TCP_TUNNEL /var/log/squid/access.log
grep TCP_HIT /var/log/squid/access.log
grep TCP_MISS /var/log/squid/access.log
grep TCP_DENIED /var/log/squid/access.log
```

# Firewall
>Firewall gui -> firewall-config
```
# reload firewall
firewall-cmd --reload

# get available zones and info
firewall-cmd  --get-zones 
firewall-cmd --info-zone=<zone name>
firewall-cmd  --list-all-zones

# get the default zone
firewall-cmd  --get–default-zone

# list active services
firewall-cmd –-list-services 
cat /etc/firewalld/zones/public.xml

# remove a service from the permanent configurations in the public zone
firewall-cmd --permanent --zone=public --remove-service=<service name>

# add rich rule to the permanent configurations in the public zone. 
# source IP can either be a subnet IP or ip address directly
firewall-cmd --permanent --zone=public --add-rich-rule='rule family=ipv4  service name=<SERVICE> source address=<IP>  accept'

# get all firewall direct rules
firewall-cmd  --direct  --get-all-rules 

# add direct rules
firewall-cmd --direct --add-rule ipv4 filter <INPUT or OUTPUT> <order> -j <DROP,ACCEPT>
firewall-cmd --direct --add-rule ipv4 filter <INPUT or OUTPUT> <order> -p <service name> -j <DROP,ACCEPT>
firewall-cmd --direct --add-rule ipv4 filter <INPUT or OUTPUT> <order> -m state --state ESTABLISHED,RELATED -j ACCEPT

# examples
firewall-cmd --direct --add-rule ipv4 filter OUTPUT 3 -m state --state ESTABLISHED,RELATED -j ACCEPT
firewall-cmd --direct --add-rule ipv4 filter OUTPUT 5 -p udp   --dport=53 -j ACCEPT
firewall-cmd --direct --add-rule ipv4 filter OUTPUT 6 -p tcp  --dport=80 -j ACCEPT
firewall-cmd --direct --add-rule ipv4 filter OUTPUT 99 -j DROP
```
> when there is connection established anything related to that connection is allowed
> order = priority 
> anything after the direct firewall rules will be accepted
> input = incoming
> output = outgoing
```

