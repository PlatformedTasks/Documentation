# FTP Configurations (for the PLAS testbed)

1. Install `vsftpd` on the server you intend to use as FTP server:
```bash
sudo apt install vsftpd
```
2. Create a user for the FTP:
```bash
sudo useradd -m ftpuser
```
3. Create the structure for the FTP directories, in our case:
```bash
sudo mkdir -p /home/ftpuser/ftp/files/out
``` 
4. Change the owner of the `/home/ftpuser/ftp` to:
```bash
sudo chown nobody:nogroup /home/ftpuser/ftp
```
5. Configure the FTP server editing the `/etc/vsftpd.conf` file. Our configuration is the following:
```
$ sudo vim /etc/vsftpd.conf

listen=NO
listen_ipv6=YES
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_file=/var/log/vsftpd.log
chroot_local_user=YES

userlist_enable=NO
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO

force_dot_files=YES
pasv_min_port=40000
pasv_max_port=50000

user_sub_token=$USER
local_root=/home/$USER/ftp
```
6. Install `nfs-common` on all the nodes:

```bash
sudo apt install nfs-common
```