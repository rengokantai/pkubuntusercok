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

#####Chapter 3. Working with Web Servers
multi-processing modules (MPM)
######Installing and configuring the Apache web server
```
apt-get install apache2 -y
add-apt-repository -y ppa:ondrej/apache2
apt-key update
apt-get update
apt-get --only-upgrade install apache2 -y
```
make a site
```
cd /var/www
mkdir example
chmod 750 example
echo 'ke' > index.html
```
create conf
```
cd /etc/apache2/sites-available
cp 000-default.conf example.conf
```
then
```
vim example.conf
```
edit
```
DocumentRoot /var/www/example
```
then
```
a2dissite 000-default.conf
a2ensite example.conf
service apache2 reload
```
config fqdn
```
vim /etc/apache2/conf-available/fqdn.conf
```
edit
```
ServerName  localhost
```
then enable
```
a2enconf fqdn
service apache2 reload
```
enable http2
```
a2enmod http2
```
```
<VirtualHost *:443>
    Protocols h2 http/1.1
    ...
</VirtualHost>
```
######Serving dynamic contents with PHP
```
apt-get update
apt-get install -y php7.0 libapache2-mod-php7.0
```
(other mods)
```
apt-get install -y php7.0 libapache2-mod-python
apt-get install -y php7.0 libapache2-mod-perl2
apt-get install -y php7.0 libapache2-mod-passenger (ruby)
```
then
```
vim /var/www/example/index.php
```
edit
```
<?php echo phpinfo(); ?>
```
then
```
vim /etc/apache2/sites-available/example.conf
```
edit
```
DirectoryIndex index.php index.html
DocumentRoot /var/www/example
```
php-config
```
vim /usr/lib/php/7.0/php.ini-development
```
use the ini file in apache
```
cd /etc/php/7.0/apache2
mv php.ini php.ini.orig
ln -s /usr/lib/php/7.0/php.ini-development php.ini
```

install lamp
```
apt-get install lamp-server^   //OR tasksel install lamp-server
```
ungrade to php7 on u14
```
apt-get install software-properties-common
add-apt-repository ppa:ondrej/php
apt-get update
apt-get install php7.0
```
######Hosting multiple websites with a virtual domain
```
a2ensite example*
a2ensite dev.example1.com.conf
service apache2 reload
```
test
```
a2query -s
```
get all avail ip addresses
```
ifconfig | grep "inet addr"
```
edit
```
Listen 80
<VirtualHost 192.1.1.1>
```
######Securing web traffic with HTTPS
```
mkdir /etc/apache2/ssl && cd /etc/apache2/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl.key -out ssl.crt
a2enmod ssl
service apache2 restart
service apache2 reload
```
then
```
<VirtualHost *:443>
SSLEngine on
SSLCertificateFile /etc/apache2/ssl/ssl.crt
SSLCertificateKeyFile /etc/apache2/ssl/ssl.key
```
```
openssl genrsa -des3 -out server.key 2048
openssl rsa -in server.key -out server.key.insecure
mv server.key server.key.secure
mv server.key.insecure server.key
openssl req -new -key server.key -out server.csr
```
now you can submit this CSR for signing purposes.
######Benchmarking
```
apt-get install apache2-utils
ab -n 10000 -c 200 -t 2 -k "http://127.0.0.1/index.php"
```












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
######Adding users and assigning access rights
```
create user ‘dbuser’@’localhost’ identified by ‘password’;
select user, host, authentication_string from mysql.user where user = ‘dbuser’; // in mysql 5.6, it is called password
grant all privileges on *.* to ‘dbuser’@’localhost’ with grant option;
show grants for ‘dbuser’@’localhost’
```
allow user login from anywhere
```
create user ‘dbuser’@’%’ identified by ‘password’;
```
drop user
```
drop user ‘dbuser’@’%’;
```
source limits
```
grant all on db.* to ‘dbuser’@’localhost’ with max_queries_per_hour 20 max_updates_per_hour 10 max_connections_per_hour 1 max_user_connections 2;
```
#####Chapter 7. Cloud Computing
######
```
apt-get install kvm cloud-utils genisoimage bridge-utils -y
```
check
```
kvm-ok
```
download ubuntu test iso
```
 wget http://cloud-images.ubuntu.com/releases/trusty/release/ubuntu-14.04-server-cloudimg-amd64-disk1.img -O trusty.img.dist
 ```
 #####Chapter 8. Working with Containers
 ######Understanding Docker volumes
 ```
 docker run -dP -v /var/lib/mysql --name mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest
 docker inspect mysql
 ```
 create a local folder and mount it as a volume inside a container at /var/lib/mysql.
 ```
mkdir ~/mysql && docker run -dP -v ~/mysql:/var/lib/mysql --name mysql mysql:latest
```
create backup
```
docker run --rm --volumes-from mysql -v ~/backup:/backup ubuntu tar cvf /backup/mysql.tar /var/lib/mysql
```

#####Chapter 10. Communication Server with XMPP
######Installing Ejabberd
```
root@a:~# wget https://www.process-one.net/downloads/downloads-action.php?file=/ejabberd/16.06/ejabberd_16.06-0_amd64.deb -O ejabberd.deb
/opt/ejabberd-16.06/bin/ejabberdctl start
/opt/ejabberd-16.06/bin/ejabberdctl status
```
create a admin
```
/opt/ejabberd-16.06/bin/ejabberdctl register admin localhost pass
```
or
```
vim /opt/ejabberd-16.06/conf/ejabberd.yml   /acl
```
login
```
admin@host  admin
```
######create users
```
./ejabberdctl register user1 a root
```
list all accounts
```
./ejabberdctl registered_users a  
```
using client cool: PSI














#####Chapter 11. Git Hosting
######Git
```
git config --list
```
######Installgitlab(require 2GB vps)
```
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
apt-get install gitlab-ce=8.9.5-ce.0
gitlab-ctl reconfigure
gitlab-ctl status
```
open browser to set new account.
######Git hook
```
touch .git/hooks/post-commit
```
edit
```
#!/bin/bash
echo "Post commit hook started"
WEBROOT=/var/www/git-hooks-demo
TARBALL=/tmp/myapp.tar
echo "Exporting repository contents"
git archive master --format=tar --output $TARBALL
mkdir $WEBROOT/html_new
tar -xf $TARBALL -C $WEBROOT/html_new --strip-components 1
echo "Backup existing setup"
mv $WEBROOT/html $WEBROOT/backups/html-'date +%Y-%m-%d-%T'
echo "Deploying latest code"
mv $WEBROOT/html_new $WEBROOT/html
exit 0
```
#####Chapter 14. Centralized Authentication Service
######Installing OpenLDAP
```
apt-get update
apt-get install slapd ldap-utils
dpkg-reconfigure slapd
```
set:
```
NO->example.com->example->(pass)->HDB->NO->YES->NO
```
test
```
ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn
ldapsearch -x -LLL -b dc=example,dc=com
```
######Installing phpLDAPadmin
```
apt-get install phpldapadmin
vim /etc/phpldapadmin/config.php
```
edit
```
$config->custom->appearance['hide_template_warning'] = true;
```

If error
```
vim /usr/share/phpldapadmin/lib/TemplateRender.php
```
edit
```
$default = $this->getServer()
->getValue('appearance','password_hash');
```
to
```
$default = $this->getServer()
->getValue('appearance','password_hash_custom');
```
