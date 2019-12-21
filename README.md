# Linux Server Configuration Project

## Server Details

IP address : 174.129.232.166

SSH port : 2200

EC2 URL : 174.129.232.166.xip.io


## Configuration steps 
### 1. Create an instance in AWS Lightsail 

First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.

Click ***`Create instance`*** and choose ***`Linux/Unix`,`OS only` `Ubuntu 18.04LTS`***

Choose a payment plan (the cheapest plan is enough for now and it's free for first month)

Click ***`Create`*** button to create an instance.

Give your instance a hostname and wait for it to start up.


### 2. SSH through Lightsail and Update all currently installed packages

When you SSH in, you'll be logged as the ubuntu user. When you want to execute commands as root, you'll need to use the sudo command to do it. To ensure the system is secure we need to keep our software up to date with new releases

- ***`sudo apt update`***    # update available package lists
- ***`sudo apt upgrade`***    # upgrade installed packages
- ***`sudo apt autoremove`*** # automatically remove packages that are no longer required


### 3. Create a new user called grader and give sudo access 

- ***`sudo adduser grader`*** # create a new user named grader

Create a new file in sudoer directory with 
- ***`sudo nano /etc/sudoers.d/grader`*** 

Add ***`grader ALL=(ALL:ALL) ALL`***  in nano editor 

### 4. Create an SSH key pair for grader using the ssh-keygen tool

Set SSH keys for grader user with `ssh-keygen` in your local machine.
- ***`ssh-keygen`***

> #Enter file in which to save the key (/home/nnnn/.ssh/id_rsa): grader
#empty passphrase


Copy the generated SSH to a virtual environment.

Run the following command in your virtual environment.

- ***`sudo mkdir /home/grader/.ssh`***

- ***`sudo chown grader:grader /home/grader/.ssh`*** # changing ownership of .ssh to grader

- ***`sudo chmod 700 /home/grader/.ssh`***           # change folder permission

- ***`sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/`***

- ***`sudo chmod 644 /home/grader/.ssh/authorized_keys`*** #and copy your generated SSH key here.

Reload SSH with:

- ***`service ssh restart`***

Now you can login as grader user.

### 5. Disable rootlogin.

Open ***`/etc/ssh/sshd_config``*** and find ***``PermitRootLogin`*** and change it to ***`no`***.


### 6. Change SSH port from 22 to 2200

- ***`sudo nano /etc/ssh/sshd_config`***  # change port 22 to 2200

- ***`sudo service ssh restart`***       # restart ssh service


### 7. Set up Uncomplicated Fire Wall (UFW)

>#### Important Notes: Go to AWS page and set up relevant ports from `networking` tab.
>•	When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server.

>•	When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used.


Configure UDW to allow only incoming request from port2200(SSH), port80 (HTTP) and port123 (NTP).

- ***`sudo ufw status `*** # utf should be inactive

- ***`sudo ufw default deny incoming`*** # deny all incoming requests

- ***`sudo ufw default deny outgoing`*** # allow all outgoing requests

- ***`sudo ufw allow 2200/tcp`*** # allow incoming ssh request

- ***`sudo ufw allow 80/tcp`*** # allow all http request

- ***`sudo ufw allow 123/udp`*** # allow ntp request

- ***`sudo ufw deny 22`*** # deny incoming request for port 22

- ***`sudo ufw enable`*** # enable ufw

- ***`sudo ufw status`*** # check current status of ufw



### 7. Set up local time zone

- ***`sudo dpkg-reconfigure tzdata`*** # choose EST

### 8. Install Apache application and wsgi module for python3

- ***`sudo apt install apache2`***                  # install apache
- ***`sudo apt install libapache2-mod-wsgi-py3`***  # install python3 mod_wsgi


- ***`sudo service apache2 start`***  # Start the server 

### 9. Install git

- ***`sudo apt install git`*** # Install git

Configure your username and email. `git config --global user.name <username>` and `git config --global user.email <email>`

### 10. Clone the project

- ***`sudo mkdir /var/www/catalog`*** # create catalog folder
- ***`sudo chown -R grader:grader catalog`***
- ***`cd /var/www/catalog`***
- ***`sudo git clone https://github.com/nnnn/fullstack-nanodegree-vm.git`***

Add `catalog.wsgi` file.

- ***`sudo nano catalog.wsgi`*** # Add `catalog.wsgi` file.

and add the following code.

```
#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'

```
- ***`sudo chown grader:grader catalog.wsgi`*** # changing ownership of catalog.wsgi to grader

Modify filenames to deploy on AWS.

- ***`mv application.py  __init__.py`*** # Rename your `application.py` to `__init__.py` 


### 11. Install virtual environment and Flask framework
Run the following command as sudo in order to install the build-essential, libssl-dev, libffi-dev and python-dev packages to your system:
- ***`sudo apt install build-essential libssl-dev libffi-dev python-dev`*** 

- ***`sudo apt install python3-pip`*** # Install `python3-pip`

- ***`sudo apt upgrade python3`*** # upgrade python3 to latest version

- ***`sudo apt install -y python3-venv`*** # Create the  virtual environment

In the catalog directory, we will be creating a new virtual environment where you can write your Python programs and create projects.

- ***`python3 -m venv catalog_venv`***  # creating a new virtual environment 
- ***`source catalog_venv/bin/activate`*** # Activate the Python Virtual Environment

- ***`sudo chmod -R 777 venv`*** # Change permissions to the viertual environment folder

Install Flask and dependencies:

- ***`sudo apt install python3-pip`***
- ***`sudo apt install python3-psycopg2`***
- ***`sudo pip3 install --upgrade pip`***
- ***`sudo pip3 install Flask`***
- ***`sudo pip3 install httplib2`***
- ***`sudo pip3 install requests`***
- ***`sudo pip3 install oauth2client`***
- ***`sudo pip3 install sqlalchemy`***
- ***`sudo pip3 install python3-psycopg2`***
- ***`sudo pip3 install bleach`***
- ***`sudo pip3 install flask_sqlalchemy`***
- ***`sudo pip3 install psycopg2-binary`***
- ***`sudo pip3 install certifi`***
- ***`sudo pip3 install --upgrade google-api-python-client oauth2client`***
- ***`python3 -m pip install flask-sqlalchemy`***
- ***`python3 -m pip install passlib`***
- ***`python3 -m pip install flask_httpauth`***
- ***`python3 -m pip install chardet`***


### 12. Configure Apache and enable a new virtual host

- ***`sudo nano /etc/apache2/sites-available/catalog.conf`*** # Create a virtual host conifg file



Paste the following code:

```
<VirtualHost *:80>
            ServerName 174.129.232.166
            ServerAlias 174.129.232.166.xip.io
            ServerAdmin admin@174.129.232.166
            WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
            WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/catalog_venv/lib/python3.6
            WSGIProcessGroup catalog
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


- ***`sudo a2ensite catalog.conf`*** # Enable the new virtual host 

### 13. Install and configure PostgressSQL

Install the PostgreSQL server along with the PostgreSQL contrib package which provides several additional features for the PostgreSQL database:

- ***`sudo apt install libpq-dev python-dev`*** # Install some necessary Python packages for working with PostgreSQL
- ***`sudo apt install postgresql postgresql-contrib`*** 

Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a trusted user who can access the database software.

- ***`sudo su - postgres`*** # Change the user
- ***`psql`*** # connect to the database system 
- ***`CREATE USER catalog WITH PASSWORD 'password';`*** # Create a new user called 'catalog' with it's password
- ***`ALTER USER catalog CREATEDB;`*** # Give *catalog* user the CREATEDB capability
- ***`CREATE DATABASE catalog WITH OWNER catalog;`*** # Create the 'catalog' database owned by *catalog* user
- ***`\c catalog`*** # Connect to the database
- ***`REVOKE ALL ON SCHEMA public FROM public;`*** # Revoke all rights
- ***`GRANT ALL ON SCHEMA public TO catalog;`*** # Lock down the permissions to only let *catalog* role create tables
- ***`\q`*** # Log out from PostgreSQL
- ***`exit`*** # Return to the *grader* user

Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:pass@localhost/catalog')
```

- ***`python3 /var/www/catalog/catalog/lotsOfCategorieswithusers.py`*** # Setup the database 

To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file:
- ***`sudo vim /etc/postgresql/10/main/pg_hba.conf`*** # Check if no remote connections are allowed with and edit it, if necessary, to make it look like this: 
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
### 14 - Update OAuth authorized JavaScript origins

To let users correctly log-in change the authorized URI to:

[ec2-174-129-232-166.compute-1.amazonaws.com](http://ec2-174-129-232-166.compute-1.amazonaws.com/) on Google developer dashboard.


### 15. Restart Apache to launch the app 

- ***`sudo service apache2 restart`*** # Restart Apache

Check `http://174.129.232.166/`

## References

[Flask document-deployment](http://flask.pocoo.org/docs/0.12/deploying/#deployment)
[Flask document](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
[Digital Ocean-mod_wsgi](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)
[DigitalOcean-postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
[DigitalOcean-apache-web-server](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04)
[iliketomatoes'repository](https://github.com/iliketomatoes/linux_server_configuration)
https://linuxize.com/post/how-to-install-postgresql-on-ubuntu-18-04/
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

#### Special thanks to [*BC Ko*](https://github.com/bcko) who wrote a really helpful README in his [repository](https://github.com/bcko/Ud-FS-LinuxServerConfig-LightSail).



## Authors

* **Manouchehr Bagheri** - *Initial work* - [Manonuro](https://github.com/Manonuro)


