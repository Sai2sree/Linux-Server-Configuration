# Linux-Server-Configuration
This web app is a project for the Udacity [FSND Course](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

## Project Overview
Took a baseline installation of a Linux server and prepared it to host my web applications. I secured my server from a number of attack vectors, installed and configured a database server, and deployed one of my existing web application onto it.

## Server details
- IP address: 34.239.233.246
- SSH Port: 2200
- Web Application URL: http://34.239.233.246/ or http://ec2-34-239-233-246.compute-1.amazonaws.com

## Configuration Steps
1. Create an instance with [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources).
2. Connect to the instance on a local machine.
3. Upgrade currently installed packages.
4. Change the SSH port from 22 to 2200.
5. Configure the firewall.
6. Create a new user named grader.
7. Give grader user sudo permissions.
8. Allow grader to log in to the virtual machine.
9. Configure the local timezone to UTC.
10. Install and configure Apache.
11. Install mod_wsgi.
12. Install PostgreSQL and ensure no remote connections are allowed.
13. Create a new PostgreSQL user named catalog with limited permissions.
14. Create a Linux user named 'catalog'.
15. Give sudo permissions to the user 'catalog'.
16. Create a new PostgreSQL database named 'catalog'.
17. Install git and clone the catalog project.
18. Setup a Google Plus auth application and add client_secrets.json file.
19. Setup a Facebook auth application and add fb_client_secrets.json file.
20. Set up a virtual environment and install dependencies.
21. Set up and enable a virtual host.
22. Write a .wsgi file.
23. Disable the default Apache site.
24. Change the ownership of the project directories.
25. Set up the database schema and populate the database.

## Step by Step Walkthrough
### 1. Create an instance with Amazon Lightsail
1. Create an Amazon Web Services account and sign in to [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources).
2. Click on 'Create an instance'.
3. Choose 'OS Only' and 'Ubuntu 16.04 LTS' options.
4. Choose a instance plan.
5. Give the instance a unique name and click 'Create'.
6. Wait for it to start up.

### 2. Connect to the instance on a local machine
1. Navigate to [Amazon Lightsail 'Account page'](https://lightsail.aws.amazon.com/ls/webapp/account/keys) and download the instance's private key.
2. 'LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem' will be downloaded. Open this in a text editor.
3. Copy the text and put it in a file called lightsail_key.rsa in the local ``~/.ssh/`` directory.
4. Run ``chmod 600 ~/.ssh/lightsail_key.rsa``
5. SSH into the server using   ``ssh -i ~/.ssh/lightsail_key.rsa ubuntu@XX.XX.XX.XX`` <br/> where XX.XX.XX.XX is the public IP address of the instance. 
- **Notes:**
  - Amazon Lightsail provides a browser-based connection method, but this will not work once the SSH port is changed.
  - Lightsail will not allow anyone to log in as root; ubuntu is the default user for Lightsail instances.
### 3. Upgrade currently installed packages
1. Notify the system of what package updates are available by running ``sudo apt-get update``.
2. Install the packages using ``sudo apt-get upgrade``.

### 4. Change the SSH port from 22 to 2200
1. Open up the /etc/ssh/sshd_config file using ``sudo nano /etc/ssh/sshd_config``.
2. Change the SSH port from 22 to 2200.
3. Restart SSH by running ``sudo service ssh restart``.

### 5. Configure the firewall
- Configure the Uncomplicated Firewall (UFW) to only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw default deny incoming 
sudo ufw default allow outgoing 
sudo ufw allow ssh 
sudo ufw allow 2200/tcp 
sudo ufw allow www 
sudo ufw allow 123/udp 
sudo ufw deny 22 
sudo ufw enable 
```
- Run `sudo ufw status` to check the open ports and to see if the ufw is active.
- From the terminal in local computer, SSH into server using <br/>
   `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`

### 6. Create a new user named *grader*
1. Create grader using  ``sudo adduser grader``
2. Enter a new UNIX password (twice), when prompted.
3. Fill out information for the new user *grader*.
4. To switch to the user *grader*, run su - grader, and enter the password.

### 7. Give grader user sudo permissions
1. Open the sudo configuration using ``sudo visudo``.
2. Search for the line ``root ALL=(ALL:ALL) ALL`` and the following line below it. <br/>
      ``grader ALL=(ALL:ALL) ALL``
3. Save and close the visudo file.

### 8. Allow grader to log in to the virtual machine
1. To generate key-pair run ``ssh-keygen`` on the local machine.
2. Give a file name for key-pair and enter the paraphrase twice, when prompted.
3. Two files will be generated. The one that ends with *.pub* is the public key.
4. Log into the virtual machine.
5. Switch to user *grader* using ``su - grader``.
6. Create a new directory called ``.ssh`` using ``mkdir .ssh``.
7. Create a file ``authorized_keys`` in ``.ssh`` directory using ``touch .ssh/authorized_keys``.
8. On the local machine, run ``cat ~/.ssh/ENTER_THE_NAME_OF_FILE.pub`` and copy the contents of the file.
9. Past them in ``.ssh/authorized_keys``file on the virtual machine.
10. Run ``chmod 700 .ssh`` on the virtual machine.
11. Run ``chmod 644 .ssh/authorized_keys`` on the virtual machine.
12. Run ``nano /etc/ssh/sshd_config`` , change 'PasswordAuthentication' to 'no', if it is 'yes'. This forces key-based authentication.
13. Run ``sudo service ssh restart``.
14. Log in as the grader using ``ssh -i ~/.ssh/ENTER_THE_NAME_OF_FILE -p 2200 grader@XX.XX.XX.XX``.

### 9. Configure the local timezone to UTC
1. Run ``sudo dpkg-reconfigure tzdata`` and follow the instructions to configure the timezone.

### 10. Install and configure Apache
1. Install Apache using ``sudo apt-get install apache2``.
2. Type your instance's public IP address in the browser. You will see the apache ubuntu default page.

### 11. Install mod_wsgi
1. Run ``sudo apt-get install libapache2-mod-wsgi python-dev``.
2. Run ``sudo a2enmod wsgi`` and ensure that mod_wsgi is enabled.

### 12. Install PostgreSQL and ensure no remote connections are allowed
1. Install PostgreSQL using ``sudo apt-get install postgresql``.
2. Run `` nano /etc/postgresql/9.5/main/pg_hba.conf`` and ensure that remote connections are not allowed.

### 13. Create a new PostgreSQL user named 'catalog' with limited permissions
1. Switch to user "postgres" using ``sudo su - postgres``.
2. Get into postgreSQL shell using ``psql``.
3. Create a user 'catalog' by using ``CREATE ROLE catalog WITH LOGIN;``.
4. Run ``ALTER ROLE catalog CREATEDB;`` to give the catalog user the ability to create databases.
5. Set a password for the user 'catalog' using ``\password catalog``.
6. Quit postgreSQL by using ``\q``.
7. Exit the user 'postgres' and switch back to user 'ubuntu'.

### 14. Create a Linux user named 'catalog'
1. Run ``sudo adduser catalog`` to create a user named 'catalog'.
2. Follow the instructions and fill out all the information for 'catalog'.

### 15. Give sudo permissions to the user 'catalog'
1. Open the sudo configuration using ``sudo visudo``.
2. Search for the line ``root ALL=(ALL:ALL) ALL`` and the following line below it. <br/>
      ``catalog ALL=(ALL:ALL) ALL``
3. Save and close the visudo file.

### 16. Create a new PostgreSQL database named 'catalog'
1. Switch to user 'catalog' by using ``su - catalog``.
2. Run ``createdb catalog`` to create a database named 'catalog'.
3. View the new database that has been created by running ``psql`` and then ``\l``.
4. Switch back to user 'ubuntu' by running ``exit``.

### 17. Install git and clone the catalog project
1. Install git using ``sudo apt-get install git``.
2. Create a directory called 'ItemCatalog' in the ``/var/www/`` directory.
3. Change to the ``ItemCatalog`` directory, and clone the project using the following command:
```
sudo git clone https://github.com/Sai2sree/Item-Catalog.git ItemCatalog
```
4. Go to ``/var/www`` directory and change the ownership of the 'ItemCatalog' directory to ubuntu using the command:
```
sudo chown -R ubuntu:ubuntu ItemCatalog/
```
5. Go to ``/var/www/ItemCatalog/ItemCatalog`` directory and change the name of project.py to \_\_init\_\_.py using following command:
``mv application.py __init__.py``.
6. In \_\_init\_\_.py, find line ``506: app.run(host='0.0.0.0', port=8000)`` and change it to ``app.run()``.
7. Switch the database in the application from SQLite to PostgreSQL. Replace the line 23 in \_\_init\_\_.py, line 64 in database_setup.py, and line 6 in populator.py with the following line:
```
engine = create_engine('postgresql://catalog:ENTER_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
```

### 18. Setup a Google Plus auth application add client_secrets.json file
1. Go to [Google Developers](https://console.developers.google.com/project) page and login with Google.
2. Create a new project
3. Name the project
4. Select "API's and Auth-> Credentials-> Create a new OAuth client ID" from the project menu
5. Select Web Application
6. On the consent screen, type in a product name and save.
7. In **Authorized Javascript origins**, add ``http://XX.XX.XX.XX`` and ``http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com``.
8. In **Authorized redirect URIs**, add the following URIs
```
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback
```
8. Click create client ID
9. Click download JSON and save it into the root directory of this project.
10. Rename the JSON file as "client_secret.json"
11. In ``/var/www/ItemCatalog/ItemCatalog/templates/login.html`` in line 26, replace the 'data-clientid' value so that it uses your Client ID from the web applciation.
12. Add the complete file path for the client_secrets.json file in lines 18 and 136 in the \_\_init\_\_.py file; change it from ``client_secrets.json`` to ``/var/www/ItemCatalog/ItemCatalog/client_secrets.json``.

### 19. Setup a Facebook auth application and add fb_client_secrets.json files
1. Go to [Facebook Developers](https://developers.facebook.com/) page and [Create an APP ID](https://auth0.com/docs/connections/social/facebook).
2. Go to your app.
3. Click on the settings and enter the 'Site URL'.
4. Click **+ Add Product** in the left column.
5. Find **Facebook Login** in the Recommended Products list and click **Set Up**.
6. Click **Facebook Login** that now appears in the left column.
7. In **Valid OAuth redirect URIs** section, add the following URIs
```
http://XX.XX.XX.XX/
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/
```
8. Create a file called fb_client_secrets.json file in the root directory of the repository.
9. Paste the following into the fb_client_secrets.json file:
```
{ "web": { "app_id": "ENTER_APP_ID_HERE", "app_secret": "ENTER_APP_SECRET_HERE" } }
```
10. In ``/var/www/ItemCatalog/ItemCatalog/templates/login.html`` in line 75, replace the 'appId' value so that it uses your APP ID from the web applciation.
11. In \_\_init\_\_.py file in line 48 & 51, change ``fb_client_secrets.json`` to ``/var/www/ItemCatalog/ItemCatalog/fb_client_secrets.json``.

### 20. Set up a vitual environment and install dependencies
1. Install pip with ``sudo apt-get install python-pip``.
2. Install virtualenv with apt-get by running ``sudo apt-get install python-virtualenv``.
3. Create a new virtual environment with ``sudo virtualenv venv``.
4. Activate the virutal environment ``.venv/bin/activate``.
5. Install the following dependencies with the virtual environment active:
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2
```
6. Deactivate the virtual environment by running ``deactivate``.

### 21. Set up and enable a virtual host
1. Create a file named ItemCatalog.conf in ``/etc/apache2/sites-available/`` using 
```
touch /etc/apache2/sites-available/
```
2. Add the following into the file:
```
<VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin chaitunagarajugari@gmail.com
		WSGIScriptAlias / /var/www/ItemCatalog/ItemCatalog.wsgi
		<Directory /var/www/ItemCatalog/ItemCatalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/ItemCatalog/ItemCatalog/static
		<Directory /var/www/ItemCatalog/ItemCatalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable the virtual host using ``sudo a2ensite ItemCatalog``
4. Run ``sudo service apache2 reload``.

### 22. Write a .wsgi file
1. Create the .wsgi file in ``/var/www/ItemCatalog`` directory and open it using following commands:
```
sudo touch ItemCatalog.wsgi
sudo nano ItemCatalog.wsgi
```
2. Add the following lines of code to the ItemCatalog.wsgi file:
```
activate_this = '/var/www/ItemCatalog/ItemCatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalog/")

from ItemCatalog import app as application
application.secret_key = 'super_secret_key'
```
3. Restart Apache using ``sudo service apache2 restart``.

### 23. Disable the default Apache site
1. Run ``sudo a2dissite 000-default.conf`` to disable the default Apache site.
2. Run ``sudo service apache2 reload``.

### 24. Change the ownership of the project directories
1. Apache runs as the ``www-data`` user, change the ownership of the project directories and files to the ``www-data`` user.
Go to ``/var/www`` directory and run the following command:
``sudo chown -R www-data:www-data ItemCatalog/``
2. After the ownership of the directories has been switched to ``www-data``, it is best to edit files as the ``www-data user``. Do this with the following command:
``sudo -u www-data nano ENTER_NAME_OF_FILE``

### 25. Set up the database schema and populate the database
1. Go to ``/var/www/ItemCatalog/ItemCatalog/`` directory.
2. Run ``. venv/bin/activate`` the virtualenv.
3. Run ``python lotsofbookswithuser.py`` to populate the database.
4. Run ``deactivate`` to deactivate the virtualenv.
5. Run ``sudo service apache2 restart`` to restart Apache again.
6. Visit site at ``http://XX.XX.XX.XX`` or ``http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com``.

## Additional Information

### To reboot the virtual machine
When you log into the virtual machine, the following prompt may appear:
```
*** System restart required ***
```
Then restart the machine using ``sudo reboot`` and SSH back into the machine.

### To Drop and Recreate a database
1. Run ``sudo apachectl stop`` to stop Apache.
2. Run ``sudo -u postgres psql``.
3. Run ``drop database catalog;`` to drop the current database.
4. Run ``create database catalog owner catalog;`` to recreate the database. 
5. Exit PostgreSQL and psql.
6. Run ``. venv/bin/activate`` to activate the virtual environment.
7. Run ``python lotsofbookswithuser.py`` to populate the database.
8. Run ``deactivate`` to deactivate the virtual environment.

## References
- Udacity's [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299) course.
- Digital Ocean tutorials
  - [How To Add, Delete, and Grant Sudo Privileges to Users](https://www.digitalocean.com/community/tutorials/how-to-add-delete-and-grant-sudo-privileges-to-users-on-a-debian-vps)
  - [An Introduction to Linux Permissions](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-permissions)
  - [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
  - [How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)
  - [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- Flask documentation
- SQLAlchemy documentation
- Udacity forums
  
  
  

