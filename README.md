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
};
```
test
```
dig -x 127.0.0.1
```
(disc)
######Hiding behind the proxy with squid
```
vim /etc/squid/squid.conf
```

```
cache_dir ufs /var/spool/squid 100 16 256
http_port 8080
visible_hostname proxy1
```

######Being on time with NTP
```
apt-get install ntp ntpdate -y         
ntpdate -s ntp.ubuntu.com
vim /etc/ntp.conf
```
edit
```

```
###### load balancing with HAProxy
```
apt-get install haproxy -y
```
set boot enable
```
vim /etc/default/haproxy
```
add
```
ENABLED=1
```

######Securing remote access with OpenVPN
```
apt-get install openvpn easy-rsa -y
```
then
```
vim /etc/openvpn/easy-rsa/vars
```
(disc)

######Discussing Ubuntu security best practices
```
adduser user
passwd user
adduser user sudo
```
edit sshd
```
port 2222
PermitRootLogin no
PasswordAuthentication no
AllowUsers user@(your-ip) user@(other-ip)
```
Optionally, install UFW
```
ufw allow from <your-IP> to any port 22 proto tcp  # memorize this syntax
ufw allow 80/tcp
ufw enable
```





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

######
```
apt-get install nginx php7.0-fpm -y
vim /etc/nginx/sites-available/default
```
edit
```
index index.php index.html index.htm;
location / {
    try_files $uri $uri/ /index.php;
}
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param QUERY_STRING    $query_string;
    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
}
```
then
```
vim /etc/php/7.0/fpm/php.ini
```
edit
```
cgi.fix_pathinfo=0
```
restart
```
service php7.0-fpm restart && service nginx restart
```
create index page
```
vim /var/www/html/index.php
```
edit
```
<?php phpinfo(); ?>
```
purge apache server
```
apt-get install python-software-properties
apt-get install software-properties-common
service apache2 stop
apt-get remove --purge apache2 apache2-utils apache2.2-bin apache2-common
```
######
```
vim /etc/nginx/sites-available/reverse_proxy
```
edit
```
```
link
```
ln -s /etc/nginx/sites-available/reverse_proxy /etc/nginx/sites-enabled/reverse_proxy
```
then
```
vim /etc/apache2/ports.conf
```
edit
```
listen 127.0.0.1:8080
```

then
```
vim /etc/apache2/sites-available/example.com
```
edit
```
<VirtualHost 127.0.0.1:8080>
  ServerName example.com
  ServerAdmin e@example.com
  DocumentRoot /var/www/example.com/public_html
</VirtualHost>
```
```
a2dissite 000-default.conf && a2ensite example.com.conf
```
then
```
service apache2 restart  && service nginx restart
```
check
```
netstat -pltn | egrep '(nginx|apache)'
```

######Setting HTTPs on Nginx
```
mkdir -p /etc/nginx/ssl/example.com
vim /etc/nginx/sites-available/example.com
```
edit
```
server {
  listen 80;
  server_name example.com www.example.com;
  return 301 https://$host$request_uri;
}
server {
  listen 443 ssl;
  server_name example.com www.example.com;

  
root /var/www/example.com/public_html;
  index index.php index.html index.htm;

  ssl on;
  ssl_certificate     /etc/nginx/ssl/example.com/server.crt;
  ssl_certificate_key     /etc/nginx/ssl/example.com/server.key;
  # if you have received ca-certs.pem from Certification Authority
  #ssl_trusted_certificate /etc/nginx/ssl/example.com/ca-certs.pem;

  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 5m;
  keepalive_timeout   70;

  
ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
  ssl_prefer_server_ciphers on;
  ssl_protocols  TLSv1.2 TLSv1.1 TLSv1;
  add_header Strict-Transport-Security "max-age=31536000";

  location / {
    try_files $uri $uri/ /index.php;
  }

  location ~ \.php$ {
    include fastcgi_params;
    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
  
}
}
```
cleanup
```
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
service nginx reload
```

######Benchmarking
```
apt-get install apache2-utils
ab -n 10000 -c 200 -t 2 -k "http://127.0.0.1/index.php"
```
######Securing the web server
check all enabled modules
```
a2query -m
```
hide identity
```
vim /etc/apache2/conf-available/security.conf
```
edit
```
ServerSignature Off
ServerTokens Prod
```
disable status module
```
a2dismod status
```
nginx
```
server_tokens off;
```
apache: disable index
```
<Directory /var/www/example.com>
  Options -Indexes
</Directory>
```

or disable globally
```
vim /etc/apache2/apache2.conf
```
disable symbolic links
```
<Directory />
  Options -FollowSymLinks
</Directory>
```
install modsecurity
```
apt-get install libapache2-modsecurity && a2enmod mod-security
```
######Troubleshooting the web server

```
apache2ctl -S
```
#####Chapter 4. Working with Mail Servers
######Sending e-mails with Postfix
```
apt-get install postfix mailutils -y
```
we set system mail name:
```
mail.example.com
```
then configure
```
vim /etc/postfix/main.cf
```
```
sendmail user@host
```
press ctrl D to send  
check:
```
mail
```
######Enabling IMAP and POP3 with Dovecot
```
apt-get install dovecot-imapd dovecot-pop3d -y
```
config
```
vim /etc/dovecot/dovecot.conf
```
add new line
```
# Enable installed protocols
protocols = pop3 pop3s imap imaps
```

config
```
vim /etc/dovecot/conf.d/10-mail.conf
```
edit
```
mail_location = mbox:~/mail:INBOX=/var/spool/mail/%u
```
If you are using Maildir as your mailbox format,
```
mail_location = maildir:~/Maildir
```
To get all enabled configurations, use the doveconf -n command:
```
doveconf -n > /etc/dovecot/dovecot.conf
```


######Mail filtering with spam-assassin
```
apt-get install spamassassin spamc
```
config postfix
```
vim /etc/postfix/master.cf
```
edit
```
smtp      inet  n       -       -       -       -       smtpd -o content_filter=spamassassin
```
(dist)
######Troubleshooting the mail server
```
tail -f /var/log/mail.log | grep "dovecot"
```
enable debug mode
```
vim /etc/dovecot/conf.d/10-logging.conf
```
edit
```
auth_verbose = yes
mail_debug = yes
```
restart
```
service dovecot restart
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


######Installing web access for MySQL
```
apt-get install phpmyadmin php-mbstring php7.0-mbstring php-gettext
service apache2 restart
```
######Setting backups
```
mysqldump --databases a b -u admin -p > alldb_backup.sql
mysqldump --all-databases -u admin -p > alldb_backup.sql
```
######Optimizing MySQL performance – queries
```
set global slow_query_log = 1;
set global slow_query_log_file = '/var/log/mysql/slow.log';
```
explicitly use or ignore index
```
select * from salaries use index (salaries) where salary between 30000 and 65000 and from_date > ‘1986-01-01’;
select * from salaries where salary between 30000 and 65000 and from_date > ‘1986-01-01’ ignore index (from_date);
```
analyze
```
select * from `employees` procedure analyse();
```


You can specify hash partitioning.
```
create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    store_id int
) partition by hash(store_id) partitions 4;
```
or alter table
```
alter table employees partition by hash(store_id) partitions 4;
```
######Optimizing MySQL performance
```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
edit
```
innodb_buffer_pool_size = 512M  # around 70% of total ram
innodb_log_file_size  = 64M
innodb_file_per_table = 1
innodb_log_buffer_size = 4M
key_buffer_size = 64M
slow_query_log = 1
slow_query_log_file = /var/lib/mysql/mysql-slow.log
long_query_time = 2
query_cache_size = 0
max_connections = 300
tmp_table_size = 32M
max_allowed_packet = 32M
log_bin = /var/log/mysql/mysql-bin.log
```
(failed, donot know which section to apply)
######Creating MySQL replicas for scaling and high availability

######Troubleshooting MySQL
```
mysqlcheck -u root -p --auto-repair --check --optimize databasename
```

#####Chapter 6. Network Storage
######Installing the Samba server
```
apt-get install samba -y
smbd --version
```
config
```
cp /etc/samba/smb.conf /etc/samba/smb.conf.orignal
vim /etc/samba/smb.conf
```
edit
```
[global]
workgroup = WORKGROUP
server string = Samba Server
netbios name = ubuntu
security = user
map to guest = bad user
dns proxy = no
[Public]
path = /var/samba/shares/public
browsable =yes
writable = yes
guest ok = yes
read only = no
create mask = 644
```
then
```
mkdir -p /var/samba/shares/public
chmod 777 /var/samba/shares/public
service smbd restart
```
######Adding users to the Samba server
```
mkdir -p /var/samba/share/smbuser && useradd -d /home/smbuser -s /sbin/nologin smbuser && smbpasswd -a smbuser && chown smbuser:smbuser /var/samba/share/smbuser
vim /etc/samba/smb.conf
```
edit
```
[Private]
path = /var/samba/shares/smbuser
browsable = yes
writable = yes
valid users = smbuser
```
enable
```
service smbd reload
smbpasswd -e smbuser //enable user
smbpasswd -d smbuser //disable user
smbpasswd -x smbuser //delete user
```
syntax:
```
valid users = smbuser,smbuser2
```
######Performance tuning the Samba server
```
vim /etc/samba/smb.conf
```
edit
```
[global]
log level = 1
socket options = TCP_NODELAY IPTOS_LOWDELAY SO_RCVBUF=131072 SO_SNDBUF=131072 SO_KEEPALIVE
read raw = Yes
write raw = Yes
strict locking = No
oplocks = yes
max xmit = 65535
dead time = 15
getwd cache = yes
aio read size = 16384
aio write size = 16384
use sendfile = true
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
create named volume
```
docker volume create --name=myvolume
docker run -v myvolume:/opt alpine sh
```
######Deploying WordPress using a Docker network
using network
```
docker network create wpnet
docker network ls
docker network inspect wpnet
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=password --net wpnet mysql
docker run --name wordpress -d -p 80:80 --net wpnet -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_PASSWORD=password wordpress
```
(dis)connect a already created container
```
docker network (dis)connect wpnet mysql
```
using --link
```
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=password mysql
docker run --name wordpress -d -p 80:80 --link mysql:mysql
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
