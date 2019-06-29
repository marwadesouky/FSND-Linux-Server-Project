# FSND Final Project: Linux Server Configuration 

## 1 - Set up AWS Lightsail
## 2 - Attach Static IP Address to lightsail instance
## 3 - Connect using SSH using the user ubuntu
## 4 - Add my public key to the /home/ubuntu/.ssh/authorized_keys file to ssh using my client
## 5 - SSH from local machine using 
```ssh ubuntu@3.220.122.27 -p 22```
## 6 - Create new user
```sudo adduser marwa```
## 7 - Change to user marwa
```sudo su - marwa```
## 8 - Create an ssh directory in /home/marwa, create an authorized_keys file and add my public key to the file
```mkdir .ssh```
```cd .ssh```
```vim authorized_keys```
Copy and paste the public key into the authorized_keys and save

## 9 - add marwa as a sudoer
switch back to ubuntu user
```sudo vim /etc/sudoers.d/marwa```
type ```marwa ALL=(ALL) NOPASSWD:ALL``` into the file, save and close the file

## 10 - Change ssh port from 22 to 2200 and disable root login

### Change SSH port 
Edit the sshd_config file
```sudo vim /etc/ssh/sshd_config```

change ```port 22``` to ```port 2200```, save and close the file 
restart ssh service
```sudo service ssh restart```
add port 2200 to lightsail instance from the network page

### Disable root login
Edit the sshd_config file
```sudo vim /etc/ssh/sshd_config```
set ```PermitRootLogin``` to ```no```

exit of the current user and ssh again using port 2200
```ssh marwa@3.220.122.27 -p 2200```
## 11 - Configure the firewall
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp (Note: Changed SSH above to port 2200)
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable
```

Check firewall status ```sudo ufw status```

#12 - Update installed packages
```sudo apt-get update```
```sudo apt-get upgrade```
```sudo apt-get dist-upgrade```

## 13 - Install apache2 and wsgi
Install Apache: ```sudo apt-get install apache2```
To make sure it works visit http://3.220.122.27 which should render an apache page that says "It works"

Install Python mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi ```


## 16 - Install postgreSQL and configure the database
Install postgreSQL: ```sudo apt-get install postgresql postgresql-contrib```
Basic Server Setup:
```sudo -u postgres psql postgres```
```\password postgres``` and enter a password

### Create a new database user
Use the following commands to create a new user
```sudo su - postgres```
```psql```
```CREATE USER catalog WITH PASSWORD 'password';```

### Limit catalog user permissions
```ALTER ROLE catalog WITH LOGIN;```
```ALTER USER catalog CREATEDB;```

Create Database:
```CREATE DATABASE catalog WITH OWNER catalog;```
Login:
```\c catalog```
Revoke all rights:
```REVOKE ALL ON SCHEMA public FROM public;```
Grant access to catalog role only:
```GRANT ALL ON SCHEMA public TO catalog;```

Exit psql and the user
```\q and exit```

Restart postgresql service
```sudo service postgresql restart```

## 15 - Install Git and clone the Item Catalog project
```sudo apt-get install git```
###  Clone Item Catalog Project in /var/www
```cd /var/www```
```sudo clone https://github.com/marwadesouky/ItemCatalog.git```
```sudo mv ItemCatalog catalog```

## 16 - Install Flask and configure the app

Install pip: ```sudo apt-get install python-pip```
Install virtualenv: ```sudo pip install virtualenv```
Create a virtual environment: ```virtualenv venv```
Activate the virtual environment: ```source venv/bin/activate```
Install Flask within the virtual env: ```sudo pip install Flask```
Install other required packages: 

```
sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo apt-get install python-psycopg2 
sudo pip install flask-seasurf
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests
sudo pip install google-api-python-client    
sudo pip install google-auth 
sudo pip install google-auth-httplib2
sudo pip install Flask-Login
```

### configure app to use postgresql instead of sqlite
Remove ```engine = create_engine('sqlite:///catalogdb.db',connect_args={
        'check_same_thread': False},echo=True)```
Add ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```

## 17 - Configure Apache and WSGI
### Configure Virtual Host
```sudo vim /etc/apache2/sites-available/catalog.conf```
Add the following into the file: 
```
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin marwadesouky96@gmail.com
	ServerName 3.220.122.27 
	ServerAlias 3.220.122.27.xip.io 
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	DocumentRoot /var/www/catalog
	<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
	Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

Enable virtual host: ```sudo a2ensite catalog```

### Configure WSGI
```cd /var/www/catalog```
```sudo vim catalog.wsgi```

Add the following to the file:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

Restart Apache: ```sudo service apache2 restart```

#### Disable nginix
```sudo service nginix stop```
```sudo systemctl disable nginx```

## 18 - Update OAuth Information for Google+ and Facebook Login
### Google+
Go to the google developers console for the project
Go to Enable and Manage APIs, then go to credentials
Select the app and add http://3.220.122.27.xip.io	to the Authorized JavaScript origins
and Authorized redirect URIs

Go to the OAuth Consent Screen and add ```xip.io``` to the Authorized domains

Download the client_secrets.json file and replace the already existing file with the new one

### Facebook 
Enforces that the website be secure so im working on add a certificate to the server
Created Certificates and working on setting up Virtual host for https port
## 19 - Grader User
Create grader user: ```sudo adduser grader```
### Add grader to sudoers
Copy the marwa sudoer file and name it grader
```sudo cp /etc/sudoers.d/marwa /etc/sudoers.d/grader```
Edit the grader file to change marwa to grader
``` sudo vim /etc/sudoers.d/grader```
The file should look like this:
```grader ALL=(ALL) NOPASSWD:ALL```

### Generate ssh keys
Switch to grader user: ```sudo su - grader```
Create ssh directory: 
```cd /home/grader```
```mkdir .ssh```

Generate ssh keys:
```ssh-keygen -t rsa```



## Refrences
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps


https://www.digitalocean.com/community/questions/sites-available-and-sites-enabled

https://www.liquidweb.com/kb/troubleshooting-please-install-available-updates-release-upgrading/