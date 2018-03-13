# Linux Server Configuration Project

## Overview
> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers
* IP Address: 18.194.102.202
* SSH port: 2200
* URL: http://ec2-18-194-102-202.eu-central-1.compute.amazonaws.com/

## How to Run the server
# 1- create a new user named grader and grant them sudo permissions .
1. log inti the remote VM as root user with: `$ ssh root@18.194.102.202`.
2. add a user called grader: `$ sudo adduser grader`.
3. Create a new file in the sudores directory: `$ sudo nano /etc/sudores.d/grader`. and put the following text inside it: _grader ALL=(ALL:ALL)ALL_, then save it by pressing ctrl+x then press y then enter
4. in order to prevent the "sudo: unable to resolve host" error, you need to edit the hosts file:
  * `$ sudo nano /etc/hosts`.
  * add this to the file: `127.0.1.1 ip-172-26-4-178`
# 2 - update all packages:
1. `sudo apt-get update`

2. `sudo apt-get upgrade`

3. install finger to check users' status: `$ apt-get install finger`
# 3- Cahnge the local timezone to UTC
1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
2. install ntp daemon ntpd for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.
### [Source](https://help.ubuntu.com/community/UbuntuTime)
# 4- configure the key-based authentication for grader user
1. on your local machine use: `$ key-gen`. to make the encryption key on your local machine, you will be able to choose where to put the key and name the key whatever you like.
2. log into the remote VM as root user with ssh and cd to /home/grader and make a new directory called .ssh : `$ sudo mkdir .ssh`.
then creat a file inside .ssh directory wih: `$ sudo touch .ssh/authorized_keys`.
3. back to your local machine, copy the contents of the encryption key you made with: `$ key-gen`, with the extension .pub, and put what you copied inside `/home/grader/.ssh/authorized_keys` you just made on the remote VM, then we will change some permissions:
  * `$ sudo chmod 700 /home/grader/.ssh`.
  * `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
  * then change the owner from root to grader : `$ sudo chown -R grader:grader .home/grader/.ssh`.
4. now you are able to log into the remote VM using ssh with the following command: `$ ssh i ~/.ssh/your-private_key grader@18.194.102.202`.
# 5- Enforce key-based authentication
1. `$ sudo nano /etc/ssh/sshd_config`. find the PasswordAithentication line and change it to no.
2. `$ sudo service ssh restart`.
# 6- change the SSH port from 22 to 2200
1. `$ sudo nano /etc/ssh/sshd_config`. Find the prt line and change it to 2200
2. `$ sudo service ssh restart`
3. now you should be able to log into the remote VM using ssh with the command: `$ ssh -i ~/.ssh/ypur-private-key -p 2200 grader@18.194.102.202`.
### [Source](http://ubuntuforums.org/showthread.php?t=1739013).
# 7- Disable ssh login for root user
1. `$ sudo nano /etc/ssh/sshd_config`, then find the PermitTootLogin and change it to no
2. `$ sudo service ssh restart`.
### [Source](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server)
# 8- Configure the Uncomplicated FireWal(UFW)
1. `$ sudo ufw allow 2200/tcp`
2. `$ sudo ufw allow 80/tcp`
3. `$ sudo ufw allow 123/udp`
4. `$ sudo ufw enable`.
#9- configure firewal to monitor and control unsuccessful log ins and to ban users
install _fsil2ban_ in order to mitigate brute force attacks by users:
1. `$ sudo apt-get update`.
2. `$ sudo apt-get install fail2ban`.
3. We need the sendmail package to send the alerts to the admin user: `$ sudo apt-get install sendmail`.
4. create a file to safely customize the fail2ban functionality: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
5. open the jail.local and edit it: `$ sudo nano /etc/fail2ban/jail.local`. set the _desemail_ field to admin user;s email address

### source: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

# 10- configure cron scripts to automatically manage package updates
1. install _unattended-upgrades_ if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. then we need to enable it, run: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`.
# 11- install Apache, mod-wsgi
1. `$ sudo apt-get install apache2`.
2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask and other python frameworks applications, to install mod-wsgi run: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. enable mod-wsgi: `$sudo a2enmod wsgi`.
4. `$ sudo service apache2 start`.
# 12- Install Git
1. `$ sudo apt-get install git `.
2. configure your username: `$ git config --global user.name <usrname>`.
3. configure your email: `$ git config -- global user.email <email>`.
# 13- Clone the catalog App repo from github
1. `$ cd /var/www/`, then make a new directory named catalog: `$ sudo mkdir catalog`.
2. change the owner of the catalog folder: `$ sudo chown -R grader:grader catalog`.
3. go inside the catalog folder you just created: `$ cd catalog`. and clone the catalog repository from github: `$ git clone https://github.com/mezo77/Catalog_app.git catalog`
4. make a catalog.wsgi file to serve the application over the mod-wsgi. the catalog.wsgi should look like this:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
5. we wiil make a new branch for deployment, go inside the repo: `$ cd /var/www/catalog/catalog/catalog` and make a new branch with: `$ git checkout -b deployment` and now we made a new branch named deployment
# 14- install irtual environment, Flask and the project's dependencies
1. install pip if not installed: `$ sudo apt-get install python-pip`.
2. install virtualenv: `$ sudo pip install virtualenv`.
3. cd into `$ cd /var/www/catalog`. then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
4. activate the virtual environment: `$ source venv/bin/activate`.
5. change permissions to the virtual environment folde: `$ sudo chmod -R 777 venv`.
6. install Flask: `$ pip install Flask`.
7. install all other project's dependencies: `$ pip install bleach httplib2 requests oauth2client sqlalchemy psycopg2`.

### [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [dabapps](https://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/)
# 15- configure and enable a new virtual host
1. create a virtual host config file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. copy and paste the following lines:
```
<VirtualHost *:80>
    ServerName 52.34.208.247
    ServerAlias ec2-52-34-208-247.us-west-2.compute.amazonaws.com
    ServerAdmin admin@52.34.208.247
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
3. enable the new virtual host: `$ sudo a2ensite catalog`.

### source [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)
# 16- install and configure postgresql
1. install some python packages first first: `$ sudo apt-get install libpq-dev python-dev`.
2. install postgresql: `$ sudo apt-get install postgresql postgresql-contrib`.
3. change the user to the default postgresql user "posgres": `$ sudo su - postgres`, then connect to the database system with: `$ psql`.
4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD '12345';`
5. give the catalog user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`
6. create the catalog database owned by catalog user: `#CREATE DATABASE catalog WITH OWNER catalog;`
7. connect to the database: `# \c catalog`
8. revoke all rights: `# REVOKE ALL ON SCHEMA oublic FROM public;`
9. Lock down the permissions to only let catalog role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`
10. log out from postgresql: `# \q`, then return to user grader: `# exit`
11. inside database_config.py and application.py make sure of the following:
```
engine = create_engine('postgresql://catalog:12345@localhost/catalog')
```
12. setup the database with: `$ sudo /var/www/catalog/catalog/database_config.py`
13. we double check that no remote connections to the database are allowed. Open the following file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf` and make sure it looks like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
### source: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
# 17- install system monitor tools
1. `$ sudo apt-get update`
2. `$ sudo apt-get install glances`
3. and to start it type: `$ glances`
### Source: [ehowstuff](http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/)
# 18-  Update OAuth authorized JavaScript origins
* o let users correctly log-in change the authorized URI to [http://ec2-18-194-102-202.eu-central-1.compute.amazonaws.com/](http://ec2-18-194-102-202.eu-central-1.compute.amazonaws.com/)
on both Google and Facebook developer dashboards.
# 19- Restart Apache
* `$ sudo service apache2 restart`.
