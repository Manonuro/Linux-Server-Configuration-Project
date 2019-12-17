# Linux Server Configuration Project

## Server Details

IP address : 174.129.232.166

SSH port : 2200

EC2 URL : http://ec2-174-129-232-166.us-west-2.compute.amazonaws.com/


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

> #Enter file in which to save the key (/home/bcko/.ssh/id_rsa): grader
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

- ***`sudo ufw enable*** # enable ufw

- ***`sudo ufw status*** # check current status of ufw



### 7. Set up local time zone

- ***`sudo dpkg-reconfigure tzdata`*** # choose UTC

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
- ***`sudo git clone https://github.com/Manonuro/fullstack-nanodegree-vm.git`***

Add `catalog.wsgi` file.

Run `sudo nano catalog.wsgi` and add the following code.

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret'
```

Modify filenames to deploy on AWS.

Rename `webserver.py` to `__init__.py` 

`mv webserver.py  __init__.py`

### 11. Install virtual environment and Flask framework

Install `pip`, `sudo apt install python-pip`

Run `sudo apt install python-virtualenv` to install virtual environment

Create a new virtuall environment with `sudo virtualenv venv` and activate it `sudo service apache2 restart`
source venv/bin/activate
Change permissions to the viertual environment folder `sudo chmod -R 777 venv`

Install Flask `pip install Flask` and dependencies `pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2.`

### 12. Configure Apache

Create a config file `sudo nano /etc/apache2/sites-available/catalog.conf`

Paste the following code

```
<VirtualHost *:80>
    ServerName Public-IP-Address
    ServerAdmin admin@Public-IP-Address
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

Enable the new virtual host `sudo a2ensite catalog`

### 13. Install and configure PostgressSQL

Run `sudo apt install PostgreSQL`

Check if no remote connections are allowed with `sudo vim /etc/postgresql/10/main/pg_hba.conf`

Login to postgress `sudo su - postgres` and `psql`

Create a new user `CREATE USER catalog WITH PASSWORD 'password'`

Create a DB called 'catalog' with `ALTER USER catalog CREATEDB` and `CREATE DATABASE catalog WITH OWNER catalog`

Connect to the DB with `\c catalog`

Revoke all rights `REVOKE ALL ON SCHEMA public FROM public`

Change a grand from public to catalog `GRANT ALL ON SCHEMA public TO catalog`

Logout from postgress and return to the grader user ` \q` and `exit`

Change the engine inside Flask application.

`engine = create_engine('postgresql://catalog:password@localhost/catalog')`

Set up the DB with `python /var/www/catalog/catalog/lotsOfCategorieswithusers.py`

### 14. Restart Apache 

Run `sudo service apache2 restart` and check `http://174.129.232.166/`

## References
[Flask document](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)

[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

[iliketomatoes'repository](https://github.com/iliketomatoes/linux_server_configuration)


•	http://flask.pocoo.org/docs/0.12/deploying/#deployment
•	http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/



## Authors

* **Manouchehr Bagheri** - *Initial work* - [Manonuro](https://github.com/Manonuro)
