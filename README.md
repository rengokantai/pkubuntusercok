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
#####Chapter 2. Networking
######Connecting to a network with a static IP
```
ifconfig -a | grep eth
```
```
ifup eth0
ifdown eth0
```
```
vim /etc/network/interfaces
```
edit from
```
auto eth0
iface eth0 inet dhcp
```
to
```
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1 
    dns-nameservers 192.168.1.45 192.168.1.46
```
(ipv6)
```
iface eth0 inet6 static
address 2001:db8::1111:22222
gateway ipv6_gateway
```
restart
```
/etc/init.d/networking restart
```
use command line
```
ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```
add a route
```
route add default gw 192.168.1.1 eth0
```
add name server:
```
vim /etc/resolv.conf
```
edit
```
nameserver 192.168.1.45
nameserver 192.168.1.46
```
verify
```
ifconfig eth0
route -n
```
reset it 
```
ip addr flush eth0
```
######Installing the DHCP server
```
apt-get install isc-dhcp-server
vim /etc/dhcp/dhcpd.conf
```
edit
```
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.150 192.168.1.200;
  option routers 192.168.1.1;
  option domain-name-servers 192.168.1.2, 192.168.1.3;
  option domain-name "example.com";
}
```
restart
```
service isc-dhcp-server restart
```
######Installing the DNS server
```
apt-get update
apt-get install bind9 dnsutils
vim /etc/bind/named.conf.options
```
edit
```
forwarders{
8.8.8.8;
8.8.4.4;
}
```
test
```
dig -x 127.0.0.1
```
(disc)
#####Chapter 5. Handling Databases
######Installing relational databases with MySQL
```
vim /etc/mysql/my.cnf
```
bind to local ip addr.  
in 14.04, mysql 5.6
```
[mysqld]
bind-address = 10.0.2.6
```
for 16.04, mysql 5.7
```
apt-get update
apt-get install mysql-server-5.7
```
configure:
```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
edit
```
[mysqld]
bind-address = 10.0.2.6
```
restart
```
service mysql restart
```
install php7.0
```
apt-get install php7.0-mysql php7.0 -y
```
######Importing and exporting bulk data
```
mysqldump -u root -p db > db_backup.sql
mysqldump -u root -p db table1 table2 > table_backup.sql
mysqldump -u root -p db | gzip > db_backup.sql.gz
```
export into csv
```
SELECT id, title, contents FROM articles
INTO OUTFILE ‘/tmp/articles.csv’
FIELDS TERMINATED BY ‘,’ ENCLOSED BY ‘”’
LINES TERMINATED BY ‘\n’;
```
import data csv
```
LOAD DATA INFILE ‘c:/tmp/articles.csv’
INTO TABLE articles
FIELDS TERMINATED BY ‘,’  ENCLOSED BY ‘”’
LINES TERMINATED BY \n IGNORE 1 ROWS;
```
import
```
mysqladmin -u admin -p create db2
mysql -u admin -p db2 < db_backup.sql
```
fetch from local:  
query.sql
```
select * from tb;
```
in-out
```
mysql -u root -p db < query.sql > output.csv
```
