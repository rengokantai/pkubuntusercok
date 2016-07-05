#### pkubuntusercok
######Creating a user account
```
adduser --disabled-password --gecos "" username
```
######Creating user accounts in batch mode
```
vim users.txt
```
edit
```
alice:password:::Alice:/home/alice:/bin/bash
```
then import
```
newusers users.txt
```
######Adding group members
note the order.
```
adduser <username> <group>
```
or
```
usermod -g <group> <username>
usermod -a -G <group1>,<group2>,<group3> <username>  #without -a, any previously assigned groups will be replaced
```
######Deleting a user account
```
deluser --remove-home alice
deluser --backup --remove-home alice  #create a alice.tar.gz in current dict
```
more options
```
deluser username group # this will remove user john from group guest
deluser --group group # this will remove a group
```
set expiredate
```
usermod --expiredate 1 username # disable username
usermod --expiredate "" username # re-enable  username
usermod -e YYYY-MM-DD username
```
######Managing file permissions
```
umask -S
```
sticky bit: When sticky bit is set, only the owner or a user with root privileges can delete a file.
```
chmod +t dir
```
######Getting root privileges with sudo
```
username ALL=(ALL:ALL)ALL # allow sudo access to username
sudo  ALL=(ALL)  ALL # allow sudo access to members of sudo
```
NOPASSWD
```
sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```
######Setting resource limits with limits.conf
```
vim /etc/security/limits.conf
```
edit
```
username  soft  cpu  0  # max cpu time in minutes
username  hard  cpu  1000 # max cpu time in minutes
```
or , unprevilege change (must under haed limit)
```
ulimit -n 4096
```
######Securing user accounts
```
vim /etc/pam.d/common-password
```
edit
```
password    [success=1 default=ignore]  pam_unix.so obscure sha512
```
to
```
password    [success=1 default=ignore]  pam_unix.so obscure sha512 minlen=8
```
and add a line
```
password requisite pam_cracklib.so ucredit=-1 lcredit=-1 dcredit=-1  ocredit=-1
```
then
```
vim /etc/adduser.conf
```
edit
```
DIR_MODE=0750
```
search other similar module
```
apt-cache search libpam-
```
other measures
```
PermitRootLogin no
PasswordAuthentication no
```
install fail2ban (watch and block repeated failed actions)
```
apt-get install fail2ban 
```
