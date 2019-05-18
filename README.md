# Linux Server Configuration

This is the final project in fulfilment of Udacity's Fullstack Nanodegree Program

The following steps walk through how to secure and setup a Linux distribution on a virtual machine and install and configure a web and database server for web application hosting. To set this up, please go through the following instructions.

To access the deployed website, visit http://54.255.238.98 or http://ec2-54-255-238-98.ap-southeast-1.compute.amazonaws.com, on port 2200.

## Amazon Lightsail Server Set Up


* If you already have an account with Amazon Lightsail, login. Otherwise, create an [Amazon Lightsail account](https://lightsail.aws.amazon.com/ls/webapp/home/resources).
* After logging in, click 'Create an Instance'.
* Select `Linux/Unix` platform
* For blueprint, select `OS Only` and `Ubuntu 16.04 LTS`
* Select an instance plan (for this project, the most afforable plan will do)
* Scroll down to name your instance and click `Create`

The instance will take a few moments for setup. Once it is ready, do take note of the public IP of your instance.

* When on the "Connect tab", there is a link to your account page (https://lightsail.aws.amazon.com/ls/webapp/account/keys).

* Here, download your private key, which is a `.pem` file

## SSH into your server instance

Within the instance you just created, you can connect to it by clicking "Connect using SSH". Alternatively, take the following steps to connect via your own SSH client:

1. Move the downloaded `LightsailDefaultKey-ap-southeast-1.pem` public key file into the local folder ~/.ssh and rename it lightsail.pem
2. In your terminal, type: `chmod 600 ~/.ssh/lightsail.pem` . This secures the public key while also making it accessible.
3. To connect to the instance via the terminal: `$ ssh -i ~/.ssh/lightsail.pem ubuntu@54.255.238.98`, where `54.255.238.98` is the public IP address of the instance.

## Server configuration

### Create a grader user account

* Log in as root user `$ sudo su -`
* Create another user 'grader' `$ sudo adduser grader`
* Create a new file in the sudoers directory. `$ sudo nano /etc/sudoers.d/grader`. In this, give the grader super permissions `grader ALL=(ALL:ALL) ALL`.
* To prevent the "sudo: unable to resolve host(none)" error, edit the hosts file `$ sudo nano /etc/hosts`. Under 127.0.0.1:localhost, add `127.0.0.1 YOUR-IP-ADDRESS`.

### Update packages
Run the following commands to update all packages and set for future updates:

* `$ sudo apt-get update`
* `$ sudo apt-get upgrade`
* `$ sudo apt-get dist-upgrade`

### Change the SSH port to access your instance
   * Open up the configuration file: `$ sudo nano /etc/ssh/sshd_config`
   * Look for port number (on line 5) and change this from `22` to `2200`
   * Save and exit using `CTRL+X`, confirm with `Y`
   * Restart SSH `$ sudo service ssh restart`

### Configure the Uncomplicated Firewall (UFW)
1. Check the current firewall status using `$ sudo ufw status`
2. Deny all incoming requests using `$ sudo ufw default deny incoming`
3. Allow all outgoings using `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www using `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP using `$ sudo ufw allow 123/udp`
7. Close access through port 22 `$ sudo ufw deny 22`
8. Enable firewall using `$ sudo ufw enable`
9. Check the current firewall status using `$ sudo ufw status`. The output should look like this:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```

10. Update the firewall configuration in your Amazon Lightsail instance by going to the Networking tab.
11. Delete default SSH port 22 and add ports 123 (UDP) and 2200(TCP) in the 'Networking' tab on Lightsail. Your settings should look like the following:
```
HTTP      TCP     80
Custom    UDP     123
Custom    TCP     2200
```
12. Exit the SSH connection: `$ exit`

### Create SSH login for grader user
Open a new(second) Terminal window (Command+N) to generate a public-private key pair.

* Input `$ ssh-keygen -f ~/.ssh/grader.rsa`
* Input `$ cat ~/.ssh/grader.rsa.pub` to read the public key. Copy its contents.

Return to your original (first) terminal window logged into Amazon Lightsail as the root user.

* Move to grader's folder `$ cd /home/grader`
* In the grader folder, create an authorized_keys file
   * `$ mkdir .ssh`
   * `$ touch .ssh/authorized_keys`
   * `$ nano .ssh/authorized_keys` and paste the public key you have copied earlier (from the second terminal window). Save.

* Change the ownership and permissions of the .ssh folder to the grader user: `$ chown -R grader.grader /home/grader/.ssh`.
* Secure your authorized_keys `$ sudo chmod 700 /home/grader/.ssh` and `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
* Restart the SSH service `$ sudo service ssh restart`
* You should now be able to login as grader user with port 2200 and the generated key pair with the following command: `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@54.255.238.98`. (note: it's no longer possible to SSH after this point through Lightsail's browser-based client)
* You will be asked for grader's password. To disable it, `$ sudo nano /etc/ssh/sshd_config`. Find the line `PasswordAuthentication` and change text to no. After this, restart ssh again: `$ sudo service ssh restart`

### Configure the timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Select `None of the above` to change the timezone to UTC.

### Disable SSH for root user
This prevents attackers from attempting with root:
* `$ sudo nano /etc/ssh/sshd_config`
* Find the `PermitRootLogin` line and edit to `no`
* Restart ssh `$ sudo service ssh restart`

## Deploying the Catalog application

### Install and configure Apache
1. While logged in as `grader`, install Apache with `$ sudo apt-get install apache2`
2. In your browser go to http://54.255.238.98. You should see a Apache2 Ubuntu Default Page if Apache has been correctly configured

### Install and configure the Python mod_wsgi file
1. Install the mod_wsgi package using `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi using `$ sudo a2enmod wsgi`
3. Restart Apache using `$ sudo service apache2 restart`
4. To check if Python is installed, use `$ python`
5. Exit with exit()

### Install PostgreSQL
1. While logged in as `grader`, install PostgreSQL `$ sudo apt-get install postgresql`
2. PostgreSQL should not allow remote connections. Open the configuration file for postgreSQL `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`.
3. You should see:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
### Configure a new PostgreSQL user, catalog
1. Switch to the `postgres` user `$ sudo su - postgres`
2. Open PostgreSQL terminal with `$ psql`
3. Create the catalog user with a password 'catalog'
```
# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
# ALTER ROLE catalog CREATEDB;
```
4. Create database with owner `catalog`:
```
# CREATE DATABASE catalog WITH OWNER catalog;
```
5. Connect to the catalog database `# \c catalog`
6. Revoke all the rights using `# REVOKE ALL ON SCHEMA public FROM public;`
7. Grant access to user catalog using `# GRANT ALL ON SCHEMA public TO catalog;`
8. List the existing roles `# \du`. The output should be like this:
```
List of roles
Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
catalog   | Create DB                                                  | {}
postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```
9. If not, run `# ALTER USER catalog LOGIN;`
10. Exit from psql using `# \q`
11. Exit from user postgres using `exit`

### Create a new Linux user 'catalog'
1. Create a new Linux user called `catalog` using `$ sudo adduser catalog`
2. Give user catalog sudo access `$ sudo visudo`
3. Search for the lines that looks like this:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
4. Below this line, give sudo privileges to catalog:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL
```
5. Save and exit the file
6. Log in as user catalog using `$ sudo su - catalog`
7. Exit from user catalog using `exit`

### Install git and clone the Catalog project
1. While logged in as grader, install git using `$ sudo apt-get install git`
2. Create directory using `$ mkdir /var/www/catalog`
3. Change to that directory and clone the catalog project from the previous GitHub project `$ sudo git clone https://github.com/shannonymous/fullstack-nanodegree-vm.git file`
4. If your catalog file is like mine; nested in a few folders deep, you can move the `catalog` project folder into the `/var/www/catalog` folder. In the /catalog project folder, using `mv '/var/www/catalog/file/vagrant/catalog' to '/var/www/catalog/'`. You can also delete the extra folders from here: `rm -rf /var/www/catalog/file`
5. Change the ownership of the catalog folder from the `/var/www` folder to grader using `$ sudo chown -R grader:grader catalog/`
6. Change to the `/var/www/catalog/catalog` directory
7. Rename the `application.py` file to `__init__.py` using `$ mv application.py __init__.py`
8. In `__init__.py`, replace the line `app.run(host='0.0.0.0', port=8000)` to `app.run()`

### Setup for deploying a Flask App on Ubuntu instance
1. Install pip using `$ sudo apt-get install python-pip`
2. Install the following packages:
```
$ sudo pip install httplib2
$ sudo pip install requests
$ sudo pip install --upgrade oauth2client
$ sudo pip install sqlalchemy
$ sudo pip install flask
$ sudo apt-get install libpq-dev
$ sudo pip install psycopg2
```

### Authenticate login through Google
1. Go to [Google Cloud Platform](https://console.cloud.google.com)
2. On the left menu, hover on `APIs & Services` and click on `Credentials`
3. Create an OAuth Client ID (under the Credentials tab), and add http://54.255.238.98 and http://ec2-54-255-238-98.ap-southeast-1.compute.amazonaws.com as authorized JavaScript origins
4. Add http://ec2-54-255-238-98.ap-southeast-1.compute.amazonaws.com/oauth2callback as authorized redirect URI
5. Download the corresponding JSON file, open it and copy its contents
6. `$ nano /var/www/catalog/catalog/client_secret.json` and replace the copied contents into this file
7. In the `templates/login.html` file, replace the `client_id` in line 23

### Setup the .conf file
1. Create the configuration file for the catalog app using `$ sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
```
<VirtualHost *:80>
        ServerName 54.255.238.98
        ServerAdmin admin@domain.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>
```
3. To enable the virtual host, run `$ sudo a2ensite catalog`
4. Restart Apache with `$ sudo service apache2 reload`

### Setup the .wsgi file
1. Create and enter the .wsgi file using `$ sudo nano /var/www/catalog/catalog.wsgi`
2. Add the following to the file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
3. Restart Apache `$ sudo service apache2 reload`

### Update the file paths (client_secrets and database)
1. With `$ nano __init__.py`,  to change the client_secrets.json line to `/var/www/catalog/catalog/client_secrets.json` as follows:
```
CLIENT_ID = json.loads( open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```
2. Ensure to look through `__ini__.py` for every instance of this change and replace as stated.
3. Update the database line in `__init__.py`, `database_setup.py`, and `sportsgen.py` with `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')` so that the correct location is referenced for catalog database

### Taking the app live
1. Generate the database files by running the following commands:
```
$ python database_setup.py
$ python sportsgen.py
```
2. Restart Apache using `$ sudo service apache2 reload`
3. Disable the default Apache page. Run `$ sudo a2dissite 000-default.conf`
3. Restart Apache using `$ sudo service apache2 reload`

The application should be live at http://54.255.238.98. If there are internal errors returned, check the Apache error logs by running `$ sudo tail -100 /var/log/apache2/error.log` and resolve the traceback call error(s) it displays.


## Folder structure
After these operations, the folder structure should look like:
```
/var/www/catalog
    |-- catalog.wsgi
    |__ /catalog             # Our Application Module
         |-- __init__.py
         |-- database_setup.py
         |-- sportsgen.py
         |-- sportswithusers.db   
         |-- /static
              |-- css
              |__ img
         |-- /templates
              |-- catalog.html
              |-- editsportitem.html
              |-- newitem.html
              |-- sportitem.html
              |-- deletesportitem.htmnl
              |-- newcategory.html
              |__ sport.html
         |-- /venv3          # Virtual Environment
```

## Resources
* Much gratitude to:
  * https://github.com/boisalai/udacity-linux-server-configuration/blob/master/README.md
  * https://github.com/juvers/Linux-Configuration
* Links
  * Official Ubuntu Documentation on [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
