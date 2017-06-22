# About
 This is my submission for Project P6: Linux Server Configuration, part of Udacity's Full Stack Web Developer Nanodegree.
# project Overview
  You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.
  
# Fundamentals of the project
  - Deploying a web application to a publicly accessible server.
  - Properly securing application ensures, application remains stable and that user’s data is safe.
  
# Updating all packages on the server
	- sudo apt-get update 
	- sudo apt-get upgrade
  
# Initial Server Setup
Step One — Root Login

    ssh root@SERVER_IP_ADDRESS
    
Step Two — Create a New User

    sudo adduser grader
    
Step Three — Root Privileges

    gpasswd -a grader sudo
    
Step Four — Add Public Key Authentication 
   - Generate a Key Pair
   
       sudo ssh-keygen
       
Assuming your local user is called "localuser", you will see output that looks like the following:
       
        Generating public/private rsa key pair.
        Enter file in which to save the key (/Users/localuser/.ssh/id_rsa): 
        
 - type /Users/localuser/.ssh/userkey path in output.  
 - This generates a private key userkey, and a public key, userkey.pub.
 - copy the content of userkey.pub in droplet server during droplet creation.
 - On the server, as the root user, enter the following command to switch to the new user
    
        sudo - grader
        
 - Create a new directory called .ssh and restrict its permissions with the following commands:
    
        sudo mkdir .ssh
        sudo chmod 700 .ssh
    
 - Now open a file in .ssh called authorized_keys with a text editor. We will use nano to edit the file:
    
        sudo nano .ssh/authorized_keys
        
   insert public key which will created earlier. 
    
 - Now restrict the permissions of the authorized_keys file with this command:
    
        sudo chmod 644 .ssh/authorized_keys
        
Step Five — Configure SSH

    nano /etc/ssh/sshd_config
    
- Next, we need to find the line that looks like this:   PermitRootLogin yes
- Modify this line to "no" like this to disable root login: PermitRootLogin no
- Disabling remote root login is highly recommended on every server!

Step Six -- Reload SSH

    service ssh restart
    
- now connect with server using key-pair 

    ssh grader@138.197.39.249 -p 2200
  
## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
  
## Change the SSH port from 22 to 2200
1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`  
 
## Configure the local timezone to UTC
1. Configure the time zone 
     
       sudo dpkg-reconfigure tzdata
       
2. It is already set to UTC.    

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache 
 
       sudo apt-get install apache2
       
2. Install mod_wsgi 

       sudo apt-get install python-setuptools libapache2-mod-wsgi
       
3. Restart Apache 
 
       sudo service apache2 restart

## Install and configure PostgreSQL
1. Install PostgreSQL 
 
       sudo apt-get install postgresql
       
2. Check if no remote connections are allowed 
  
       sudo vim /etc/postgresql/9.3/main/pg_hba.conf
       
3. Login as user "postgres" 

       sudo su - postgres
       
4. Get into postgreSQL shell

       psql

5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	   postgres=# CREATE DATABASE catalog;
	   postgres=# CREATE USER catalog;
	
5. Set a password for user catalog
	
	   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	
6. Give user "catalog" permission to "catalog" application database
	
	   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	    
7. Quit postgreSQL 
 
       postgres=# \q
         
8. Exit from user "postgres" 

	   exit
     
# Prepare to deploy your project	
- install git `sudo apt-get install git`
- Use `cd /var/www` to move to the /var/www directory
- Create the application directory `sudo mkdir FlaskApp`
- Move inside this directory using `cd FlaskApp`
- Clone the Catalog App to the virtual machine `git clone https://github.com/ashishchopra605/Item-Catalog.git`
- Rename the project's name `sudo mv ./Item_Catalog ./FlaskApp`
- Move to the inner FlaskApp directory using `cd FlaskApp`
- Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
- Edit `database_setup.py`,`db.py` and change `engine = create_engine('sqlite:///newswebapplication.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
- Install pip `sudo apt-get install python-pip`
- Use pip to install dependencies `sudo pip install -r requirements.txt`
- Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
- Create database schema `sudo python database_setup.py` 

# Configure and Enable a New Virtual Host
- Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
- Add the following lines of code to the file to configure the virtual host. 
	```
	<VirtualHost *:80>
		ServerName 138.197.39.249
		ServerAdmin admin@138.197.39.249
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/models/static
		<Directory /var/www/FlaskApp/FlaskApp/models/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
- Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

# Create the .wsgi File
- Create the .wsgi File under /var/www/FlaskApp: 

	  cd /var/www/FlaskApp
	  sudo nano flaskapp.wsgi 
	
- Add the following lines of code to the flaskapp.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```
  
# Restart Apache
- Restart Apache `sudo service apache2 restart `
      

