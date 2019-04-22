# Project 5 of the Udacity Full Stack Nanodegree programme
_Configuring a Linux server to host a web app securely._

# Server details



## Check and update all currently installed packages

`sudo apt-get update` - to update the package indexes

`sudo apt-get upgrade` - to actually upgrade the installed packages

`sudo reboot` - to reboot, because there will be a reboot msg.


# Configuration changes
## Add user and set a password
Add user `grader` with command:

```
sudo adduser grader
```

## Add user grader to sudo group
```
usermod -aG sudo grader
```

## Set-up SSH keys for user grader
Become the grader user by typing:

```
$ su - grader
```

As grader user do:
```

$ mkdir .ssh
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```

I pasted the ssh key in the authorized key into the authorized_keys file.

```
chmod 700 .shh
chmod 644 .ssh/auhtorized_keys
```

now you can log in via ssh with: 

```
ssh grader@34.203.190.7 -i [privateKeyFileName]
password:student
```

##Change SSH port from 22 to 2200
Switch to root user by typing:
```
sudo su -
```

Edit the file `/etc/ssh/sshd_config` and add `Port 2200` under `Port 22`

by typing:
```
nano /etc/ssh/sshd_config
```
while we are at it let's disable root login


Then restart the SSH service:

`sudo service ssh restart`

Will now need to use the following command to login to the server:

`$ ssh grader@34.203.190.7 -p 2200 -i [privateKeyFileName]`

## Change timezone to UTC
change the time zone to UTC by typing:
`sudo timedatectl set-timezone UTC`

## Install Apache to serve a Python mod_wsgi application
Install Apache:

`sudo apt-get install apache2`

Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure PostgreSQL
Install PostgreSQL with:

`sudo apt-get install postgresql postgresql-contrib`

To ensure that remote connections to PostgreSQL are not allowed, I checked
that the configuration file `/etc/postgresql/9.3/main/pg_hba.conf` only
allowed connections from the local host addresses `127.0.0.1` for IPv4
and `::1` for IPv6.

Create a PostgreSQL user called `catalog` with:

`sudo -u postgres createuser -P catalog`

You are prompted for a password. This creates a normal user that can't create
databases, roles (users).

Create an empty database called `catalog` with:

`sudo -u postgres createdb -O catalog catalog`
