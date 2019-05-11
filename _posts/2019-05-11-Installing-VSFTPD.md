---
layout: post
title: Installing VSFTPD
---

sudo apt get vsftpd

Create (as root) `vsftpd.chroot_list` file in `/etc/`
```
cd /etc
touch vsftpd.chroot_list
```

Create new user and set her with default home directory.

```
sudo useradd myuser
sudo passwd myuser
sudo usermod --home /home/ftp myuser
```

Restart the service
```
sudo service vsftpd restart
```


My vsftpd.conf file:
```
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES

local_root=/home/ftp
local_umask=002
chmod_enable=YES
file_open_mode=0755
allow_writeable_chroot=YES
port_enable=YES
ftp_data_port=20
seccomp_sandbox=NO


dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
chroot_list_enable=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=ftp
ssl_enable=NO
```

Test your ftp server.

I am using `ncftp`.
```
ncftp -u myuser 127.0.0.1
```
