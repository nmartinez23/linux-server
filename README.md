# Linux Server Configuration

## Resources
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* AWS Lightsail
* PostgreSQL
* Apache2
* Git
* Python

The following steps detail how to connect to an AWS Lightsail virtual private server with Ubuntu Linux. 

* Start a Ubuntu Linux server instance with [Amazon Lightsail](https://amazonlightsail.com/).
* SSH into the server by clicking the "Connect" button in the Lightsail dashboard.
* A Lightsail terminal will pop up. Update all currently installed packages with the command "sudo apt-get update"
* Configure Lightsail firewall and ports to allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123). Use the following commands: **'sudo ufw status', 'sudo ufw default deny incoming', 'sudo ufw default allow outgoing', 'sudo ufw allow ssh', 'sudo ufw allow 2200/tcp', 'sudo ufw allow www', 'sudo ufw deny 22', 'sudo ufw enable'**.
* Create a 'Custom' TCP port 2200 in your Lightsail dashboard under the Networking tab.
* Close the Lightsail terminal as you will set up SSH to Lightsail in your local terminal.
* Download your private key “LightsailDefaultPrivateKey.pem” from your Lightsail dashboard.
* Open a terminal window and CD into .ssh from your root directory.
* Move your downloaded key with the command: **mv ~/Downloads/LightsailDefaultPrivateKey.pem .**.
* Change the permission on the key file with the command **chmod 600 LightsailDefaultPrivateKey.pem**
* SSH into the server as the user Ubuntu with the following command: **ssh ubuntu@Your-IP-Address -i ~/.ssh/LightsailDefaultPrivateKey.pem** You can find your IP address in the Lightsail dashboard.
* Use the command **sudo adduser grader** to create a new user. Enter in a password you will remember after this command.
* Use the command **finger grader** to ensure a new user was created.
* Give grader sudo permissions with the command **sudo visudo**.
* Open a second terminal and cd into .ssh from your root directory.
* Use the command **ssh-keygen** to generate SSH keys for grader.
* You will be prompted to save the key to a file and enter the grader's password. Use the file location provided in the prompt to save the key except you will replace 'id_rsa' with 'graderLightsail'.
* In your first terminal logged in as user Ubuntu, use the following command **sudo login grader**.
* Make a directory **mkdir .ssh** and the next command to create a file **touch .ssh/authorized_keys**.
* In the second terminal, use the command **cat graderLightsail.pub** and copy the ouput.
* Back in the first terminal, use the command **sudo nano .ssh/authorized_keys** to paste and save the file.
* Modify permissions for the .ssh directory and authorized_keys file with the commands: **chmod 700 .ssh** and then **chmod 664 .ssh/authorized_keys**.
* In your second terminal, SSH into Lightsail as grader with the command: **ssh grader@Your-IP-Address -i ~/.ssh/your-ssh-public-key-filename**.
* In the first window use the command **exit** to logout as grader and back as the Ubuntu user. Keep this window open.
* In the second window where grader is logged in, use the command **sudo nano /etc/ssh/sshd_config** to modify the file.
* In the file, change Port 22 to 2200 and then set PermitRootLogin to no. Save the file.
* Restart the server with the command **sudo service ssh restart**.
* Use the command **ssh grader@Your-IP-Address -i ~/.ssh/your-ssh-public-key-filename -p 2200** to SSH into the server as grader on port 2200.
* Set the timezone with the command **sudo dpkg-reconfigure tzdata** and select 'UTC'.
* Install Apache and mod_wsgi app with the command **sudo apt-get install apache2** and **sudo apt-get install libapache2-mod-wsgi** and **sudo a2enmod wsgi** and **sudo service apache2 start**.
* Install PostgreSQL with **sudo apt-get install postgresql postgresql-contrib** and **sudo su - postgres**.
* To start PostgreSQL: **psql** and **CREATE USER catalog WITH PASSWORD 'catalog';** and **ALTER USER catalog CREATEDB;** and **CREATE DATABASE catalog WITH OWNER catalog;**.
* To connect: **\c catalog** and **REVOKE ALL ON SCHEMA public FROM public;** and **GRANT ALL ON SCHEMA public TO catalog;**.
* To disconnect: **\q** and **exit**.
* Install Git with **sudo apt-get install git**.
* Create new directory with **cd /var/www** and **sudo mkdir catalog** and **sudo chown -R grader:grader catalog**.
* Clone repo with **git clone https://github.com/nmartinez23/item-catalog.git catalog**.
* Install Flask and Virtual Environment with **sudo apt-get install python-pip** and **sudo pip install virtualenv** and **sudo virtualenv venv** and **source venv/bin/activate**. Change permission to **sudo chmod -R 777 venv**. Then run **sudo pip install Flask** and **sudo pip install httplib2 requests oauth2client sqlalchemy psycopg2 sqlalchemy_utils**.
* Change client with `sudo nano __init__.py` and save the following in the file:
```
CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```
* Configure the virtual host with **sudo nano /etc/apache2/sites-available/catalog.conf** and save the following in the file:
```
<VirtualHost *:80>
    ServerName 52-201-102-170
    ServerAlias ec2-52-201-102-170.us-east-1a.compute.amazonaws.com
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
* Start the virtual host with **sudo a2ensite catalog** and **sudo service apache2 restart**
* **cd ..** to move back to the main catalog directory
* Create wsgi file with **sudo nano catalog.wsgi** and save the following in the file:

```
#!/usr/bin/python
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
* Restart Apache with **sudo service apache2 restart**.
* Sudo nano into your __init__.py and database_setup.py and add the following to each file:
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
* Restart with `sudo service apache2 restart` and `sudo python __init__.py`.
* Site should load at http://52.201.102.170/
* http://ec2-52-201-102-170.compute-1.amazonaws.com/
* SSH Port 2200
* HTTP web server Port 80
