# Linux Server Configuration Project
# Server Details
IP address : 3.122.237.112

SSH port : 2200

EC2 URL : ec2-3-122-237-112.eu-central-1.compute.amazonaws.com

Configuration steps
1. Create an instance in AWS Lightsail
Go to AWS Lightsail and create a new account / sign in with your account.

Click Create instance and choose Linux/Unix,OS only Ubuntu 18.04LTS

Choose the cheapest plan it's free for first month

Click Create button to create an instance.

2. Set up SSH key
Go to account page from your AWS account. You will find your SSH key there.

Download your SSH key, the file name will be like LightsailDefaultPrivateKey-*.pem

Navigate to the directory where your file is stored in your terminal.

`Run chmod 600 LightsailDefaultPrivateKey-*.pem` to restrict the file permission.

Change name to lightsail_key.rsa.

Run a command `ssh -i lightsail_key.rsa ubuntu@3.122.237.112` in your terminal.

3. Change SSH port from 22 to 2200
Edit /etc/ssh/sshd_config file by `sudo nano /etc/ssh/sshd_config`

Change port from 22 to 2200

Save the change by Control + X and exit from nano with Y

Restart SSH with `sudo service ssh restart`

4. Set up Uncomplicated Fire Wall (UFW)
Configure UDW to allow only incoming request from port2200(SSH), port80 (HTTP) and port123 (NTP).

`sudo ufw status` -- utf should be inactive

`sudo ufw default deny incoming` -- deny all incoming requests
`sudo ufw default deny outgoing`-- allow all outgoing requests

`sudo ufw allow 2200/tcp` -- allow incoming ssh request

`sudo ufw allow 80/tcp` -- allow all http request

`sudo ufw allow 123/udp` -- allow ntp request

`sudo ufw enable` -- enable ufw

Go to AWS page and set up relevant ports from networking tab.

5. Create a new user called grader and give an access
Run `sudo adduser grader` to create a new user called grader

Create a new directory in sudoer directory with `sudo nano /etc/sudoers.d/grader`

Add `grader ALL=(ALL:ALL) ALL` in nano editor

Run `sudo nano /etc/hosts`

Set SSH keys for grader user with ssh-keygen in your local machine.

Copy the generated SSH to a virtual environment.

Run the following command in your virtual environment.

`su - grader`

`mkdir .ssh`

`touch .ssh/authorized_keys`

`nano .ssh/authorized_keys` and copy your generated SSH key here.

Reload SSH with `service ssh restart`

Then now you can login grader user.

Disable rootlogin.

Open `/etc/ssh/sshd_config` and find `PermitRootLogin` and change it to no.

6. Update all packages
Run `sudo apt-get update && sudo apt-get upgrade`

7. Set up local time zone
Run `sudo dpkg-reconfigure tzdata` and choose UTC

8. Install Apache application and wsgi module
Run `sudo apt-get install apache2` to install apache

Run `sudo apt-get install python-setuptools libapache2-mod-wsgi to install mod-wsgi module`

Start the server `sudo service apache2 start`

9.Configure your git username and email. `git config --global user.name <username>` and `git config --global user.email <email>`

10. Clone the project
Run `cd /var/www and sudo mkdir catalog`

Change the owner to grader `sudo chown -R grader:grader catalog`

Switch to the catalog directory and clone the Catalog project.

cd catalog and `sudo git clone <you_flask_app_repos>`

Add catalog.wsgi file.

Run `sudo nano catalog.wsgi` and add the following code.
```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret'
```

Rename application.py to __init__.py

`mv application.py __init__.py`

11. Install virtual environment and Flask framework
Install pip, `sudo apt-get install python-pip`

`Run sudo apt-get install python-virtualenv` to install virtual environment

Create a new virtuall environment with `sudo virtualenv venv `and activate it source `venv/bin/activate`

Change permissions to the viertual environment folder `sudo chmod -R 777 venv`

Install Flask pip install Flask and dependencies `pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`.

12. Configure Apache
Create a config file sudo nano `/etc/apache2/sites-available/catalog.conf``

Paste the following code
```shell
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

13. Install and configure PostgressSQL
Run `sudo apt-get install PostgreSQL`

Remote connections to psql are desabled by default

Login to postgress `sudo su - postgres` and `psql`

Create a new user `CREATE USER catalog WITH PASSWORD 'password'`

gir user permission to Create a DB ' with `ALTER USER catalog CREATEDB` and `CREATE DATABASE catalog WITH OWNER catalog`

Connect to the DB with `\c catalog`

`Revoke all rights REVOKE ALL ON SCHEMA public FROM public`

Change a grand from public to catalog `GRANT ALL ON SCHEMA public TO catalog`

Logout from postgress and return to the grader user \q and exit

Change the engine inside Flask application, database_setup and lotsofitem files to.

```python
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

Set up the DB with `python /var/www/catalog/item-catalog-udacity/database_setup.py`

14. Restart Apache
Run `sudo service apache2 restart` and check http://3.122.237.112
