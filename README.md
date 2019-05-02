# Project 5 of the Udacity Full Stack Nanodegree program
_Configuring a Linux server to host a web app securely._

# Server details

IP address: 52.23.186.77

SSH port: 2200

Passphrase for Key: student


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
I genereated a paried ssh key file on my local machine with `ssh-keygen`
I pasted the ssh key from my public key file key into the authorized_keys file on my AWS linux server.
I then changed the access permission on this file by typing:

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
We're going to configure our firewall to only allow SSH from port 2200 instead of 22. We're also going to allow https through port 80.

```
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 2200/tcp
ufw allow www
```

Then we'll turn on our firewall with:

```
ufw enable
```

But we also have to disable through AWS dashboard

![firewall](https://i.imgur.com/drGwo5v.png)

Then restart the SSH service:

`sudo service ssh restart`

Will now need to use the following command to login to the server:

`$ ssh grader@grader@52.23.186.77  -p 2200 -i [privateKeyFileName]`


## Install Apache to serve a Python mod_wsgi application
Install Apache:

`sudo apt-get install apache2`

Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure PostgreSQL
Install PostgreSQL with:

`sudo apt-get install postgresql postgresql-contrib`

In order for make to make changes to the database as the psql user catalog I had to make some changes to a file 
`/etc/postgresql/9.5/main/pg_hba.conf`

from

```
# TYPE DATABASE USER ADDRESS METHOD
local  all      all          peer
```
to
```
# TYPE DATABASE USER ADDRESS METHOD
local  all      all          md5
```

Create a PostgreSQL user called `catalog` with:

`sudo -u postgres createuser -P catalog`

Create an empty database called `catalog` with:

`sudo -u postgres createdb -O catalog catalog`

Set a password for user catalog with:

`postgres=# ALTER ROLE catalog WITH PASSWORD 'password';`

Give user "catalog" permission to "catalog" application database

`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

Press ctrl^d to exit out of psql and then one more time to exit out of user postgres

## Installing git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `sudo git clone https://github.com/aznprox/catalog_project_for_aws`
6. Rename the project's name `sudo mv ./catalog_project_for_aws ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Install pip `sudo apt-get install python-pip`
9. Use pip to install dependencies `sudo pip install -r requirements.txt`
10. Populate our database with `python lotsofcars.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 52.23.186.77
        ServerAlias 52.23.186.77.*.xip.io
		ServerAdmin contact@talyhuang.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps