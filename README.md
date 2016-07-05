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
