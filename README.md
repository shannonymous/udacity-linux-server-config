# Linux Server Configuration

This is the final project in fulfilment of Udacity's Fullstack Nanodegree Program

The following steps walk through how to secure and setup a Linux distribution on a virtual machine and install and configure a web and database server for web application hosting. To set this up, please go through the following instructions.

To access the deployed website, visit http://54.255.238.98 or http://ec2-54-255-238-98.ap-southeast-1.compute.amazonaws.com, on port 2200.

## Amazon Lightsail Server Set Up


If you already have an account with Amazon Lightsail, login. Otherwise, create an Amazon Lightsail account.

After logging in, click 'Create an Instance';

Select Linux/Unix platform

For blueprint, select OS Only and Ubuntu

Select an instance plan (for this project, the most afforable plan will do)

Scroll down to name your instance and click 'Create'

The instance will take a few moments for setup. Once it is ready, do take note of the public IP of your instance.

When on the "Connect tab", there is a link to your account page (https://lightsail.aws.amazon.com/ls/webapp/account/keys).

Download your private key, which is a `.pem` file

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
* You should now be able to login as grader user with port 2200 and the generated key pair with the following command: `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@54.255.238.98`.
* You will be asked for grader's password. To disable it, `$ sudo nano /etc/ssh/sshd_config`. Find the line `PasswordAuthentication` and change text to no. After this, restart ssh again: `$ sudo service ssh restart`





### Configure the timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Select `None of the above` to change the timezone to UTC.
