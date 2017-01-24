<h1>Linux Configuration Project</h1>
Udacity Fullstack Nanodegree
January 19th, 2017

The goal is to take a baseline installation of a Linux distribution on a virtual machine and prepare it to host the Item Catalog web application from Item Catalog Project, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

Item Catalog Website can be accessed <a href="http://35.162.219.188/">here</a>

<h3>Configurations and Software Installed</h3>
All done in root user. If not done in root user then need to use sudo in front of commands.

<h4>Add user `grader`</h4>
`sudo adduser grader`

<h4>Give `grader` sudo permission</h4>
Add grader to sudo group (<a href="https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/ch02s03.html">citation</a>):

```usermod -aG sudo grader```

Resolve host issue (<a href="http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none">citation</a>):

Add `ip-10-20-9-8` to end of `127.0.0.1 localhost` in /etc/hosts

`reboot`

Now grader can login and use sudo

<h4>Set up SSH keys for user grader</h4>
Create .ssh file:

`mkdir /home/grader/.ssh`

Create copy authorized_keys file from root:

`cp /home/root/.ssh authorized)keys /home/grader/.ssh/authorized_keys`

Set permissions for .ssh directory and authorized_key file:

`chmod 700 .ssh`

`chmod 644 .ssh/authorized_keys`

<h4>Update all installed packages</h4>
Update package indexes:

`apt-get update`

Upgrade installed packages

`apt-get upgrade`

<h4>Change SSH port from 22 to 2200</h4>
Change line Port 22 to Port 2200 in /etc/ssh/sshd_config

Restart SSH service:

```service ssh restart```

Now need to add -p 2200 to log in

<h4>Configure Uncomplicated Firewall (UFW)</h4>
Block all incoming connections on all ports:

```ufw default deny incoming```

Allow outgoing connection on all ports:

```ufw default allow outgoing```

Allow incoming connection for SSH on port 2200:

```ufw allow 2200/tcp```

Allow incoming connections for HTTP on port 80:

```ufw allow www```

Allow incoming connection for NTP on port 123:

```ufw allow ntp```

To check the rules that have been added before enabling the firewall use:

```ufw show added```

To enable the firewall, use:
```ufw enable```

To check the status of the firewall, use:
```ufw status```

<h4>Configure the local timezone to UTC</h4>
```sudo dpkg-reconfigure tzdata```
Select local timezone accordingly

<h4>Install Apache to serve a Python mod_wsgi application</h4>
```
apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
```

<h4>Install PostgreSQL</h4>
Instal PostgreSQL (<a href="https://help.ubuntu.com/community/PostgreSQL">citation</a>):
```apt-get install postgresql postgresql-contrib```

<h4>Configure PostgreSQL</h4>
Disallow remote connections (<a href="https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps">citation</a>): 
Check `/etc/postgresql/9.3/main/pg_hba.conf` to make sure host IP connects to localhost, if not then
change IPv4 local connections to 
```host    all             all             127.0.0.1               md5```
and IPv6 local connections to 
```host    all             all             ::1               md5```
Create PostgreSQL user named catalog (<a href="https://help.ubuntu.com/community/PostgreSQL">citation</a>):
```sudo -u postgres createuser -P catalog```
Give user catalog ownership of database catalog:
```sudo -u postgres createdb -O catalog catalog```
Restart server:
```sudo /etc/init.d/postgresql reload```

<h4>Install Flask and SQLAlchemy</h4>
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
```

<h4>Install Git</h4>
```sudo apt-get install git```

<h4>Clone Item Catalog App From Github</h4>
Create directory to store Item Catalog App:
```mkdir /var/www/html/public_html/ItemCatalog```
Set permissions for item-catalog directory so www-data can access:
```chown www-data:www-data /var/www/html/public_html/ItemCatalog/```
Clone Item Catalog:
```sudo -u www-data git clone https://github.com/zeychen/UdacityFullstackItemCatalogProject.git /var/www/html/public_html/ItemCatalog```

<h4>Setup Item Catalog App (<a href="https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps">citation</a>)</h4>
In /var/www/html/public_html/ItemCatalog/ directory:
Create `__init__.py`

Change `engine = create_engine('sqlite:///catagories.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')` in `items_webserver.py`, `items_db_query.py`, and `items_db_setup.py`

Change `app.run(host='0.0.0.0', port=5000)` to `app.run()` in `items_webserver.py`

Take `app.secret_key` out of `if __name__ == __main__:` in `items_webserver.py`

Insert `/var/www/html/public_html/ItemCatalog/` in front of all `client_secrets.json` file locations within `items_webserver.py`


Password not shown for security reasons

<h4>Update Google OAuth</h4>
Change `localhost` in client_secrets.json to `http://35.162.219.188`
Change Authorized JavaScript origins in Google Developers Console

<h4>Configure Apache to use items_webserver.wsgi</h4>
Add the following after <VirtualHost> and right before </VirtualHost> (<a href="http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/">citation</a>):
```
WSGIDaemonProcess catalog user=www-data group=www-data threads=5 home=/var/www/html/public_html/ItemCatalog/items_webserver.wsgi
WSGIScriptAlias / /var/www/html/public_html/ItemCatalog/items_webserver.wsgi
<Directory /var/www/html/public_html/ItemCatalog/>
    Order allow,deny
    Allow from all
</Directory>
Alias /static /var/www/html/public_html/ItemCatalog/static
<Directory /var/www/html/public_html/ItemCatalog/static/>
    Order allow,deny
    Allow from all
</Directory>
```

<h4>Create .wsgi File </h4>
In /var/www/html/public_html/ItemCatalog/ directory:
Create items_webserver.wsgi and add
```
import sys

sys.path.append('/var/www/html/public_html/ItemCatalog/')

from items_webserver import app as application
import items_db_setup
import items_db_query
```








