<h1>Linux Configuration Project</h1>
Udacity Fullstack Nanodegree
January 19th, 2017

The goal is to take a baseline installation of a Linux distribution on a virtual machine and prepare it to host the Item Catalog web application from Item Catalog Project, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

Item Catalog Website can be accessed <a href="http://35.162.219.188/">here</a>

<h2>Configurations and Software Installed</h2>
All done in root user. If not done in root user then need to use sudo in front of commands.

<h3>Add user `grader`</h3>
`sudo adduser grader`

<h3>Give `grader` sudo permission</h3>
Add grader to sudo group (<a href="https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/ch02s03.html">citation</a>):

```usermod -aG sudo grader```
Resolve host issue (<a href="http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none">citation</a>): <br>
Add `ip-10-20-9-8` to end of `127.0.0.1 localhost` in /etc/hosts<br>
`reboot`<br>
<br>
Now grader can login and use sudo


<h3>Set up SSH keys for user grader</h3>
Create .ssh file:<br>
`mkdir /home/grader/.ssh`<br>
Create copy authorized_keys file from root:<br>
`cp /home/root/.ssh authorized)keys /home/grader/.ssh/authorized_keys`<br>
Set permissions for .ssh directory and authorized_key file:<br>
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```


<h3>Update all installed packages</h3>
Update package indexes:<br>
`apt-get update`<br>
Upgrade installed packages<br>
`apt-get upgrade`

<h3>Change SSH port from 22 to 2200</h3>
Change line Port 22 to Port 2200 in /etc/ssh/sshd_config<br>
Restart SSH service:<br>
```service ssh restart```
Now need to add -p 2200 to log in<br>

<h3>Configure Uncomplicated Firewall (UFW)</h3>
Block all incoming connections on all ports:<br>
```ufw default deny incoming```<br>
Allow outgoing connection on all ports:<br>
```ufw default allow outgoing```<br>
Allow incoming connection for SSH on port 2200:<br>
```ufw allow 2200/tcp```<br>
Allow incoming connections for HTTP on port 80:<br>
```ufw allow www```<br>
Allow incoming connection for NTP on port 123:<br>
```ufw allow ntp```<br>
To check the rules that have been added before enabling the firewall use:<br>
```ufw show added```<br>
To enable the firewall, use:<br>
```ufw enable```<br>
To check the status of the firewall, use:<br>
```ufw status```

<h3>Configure the local timezone to UTC</h3>
```sudo dpkg-reconfigure tzdata```<br>
Select local timezone accordingly

<h3>Install Apache to serve a Python mod_wsgi application</h3>
```
apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
```

<h3>Install PostgreSQL</h3>
Instal PostgreSQL (<a href="https://help.ubuntu.com/community/PostgreSQL">citation</a>):<br>
```apt-get install postgresql postgresql-contrib```

<h3>Configure PostgreSQL</h3>
Disallow remote connections (<a href="https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps">citation</a>): <br>
Check `/etc/postgresql/9.3/main/pg_hba.conf` to make sure host IP connects to localhost, if not then<br>
change IPv4 local connections to <br>
```host    all             all             127.0.0.1               md5```<br>
and IPv6 local connections to <br>
```host    all             all             ::1               md5```<br>
Create PostgreSQL user named catalog (<a href="https://help.ubuntu.com/community/PostgreSQL">citation</a>):<br>
```sudo -u postgres createuser -P catalog```<br>
Give user catalog ownership of database catalog:<br>
```sudo -u postgres createdb -O catalog catalog```<br>
Restart server:<br>
```sudo /etc/init.d/postgresql reload```

<h3>Install Flask and SQLAlchemy</h3>
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
```

<h3>Install Git</h3>
```sudo apt-get install git```

<h3>Clone Item Catalog App From Github</h3>
Create directory to store Item Catalog App:<br>
```mkdir /var/www/html/public_html/ItemCatalog```<br>
Set permissions for item-catalog directory so www-data can access:<br>
```chown www-data:www-data /var/www/html/public_html/ItemCatalog/```<br>
Clone Item Catalog:<br>
```sudo -u www-data git clone https://github.com/zeychen/UdacityFullstackItemCatalogProject.git /var/www/html/public_html/ItemCatalog```

<h3>Setup Item Catalog App (<a href="https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps">citation</a>)</h3>
In /var/www/html/public_html/ItemCatalog/ directory:<br>
Create `__init__.py`<br>
Change `engine = create_engine('sqlite:///catagories.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')` in `items_webserver.py`, `items_db_query.py`, and `items_db_setup.py`<br>
Change `app.run(host='0.0.0.0', port=5000)` to `app.run()` in `items_webserver.py`<br>
Take `app.secret_key` out of `if __name__ == __main__:` in `items_webserver.py`<br>
Insert `/var/www/html/public_html/ItemCatalog/` in front of all `client_secrets.json` file locations within `items_webserver.py`<br>

Password not shown for security reasons

<h3>Update Google OAuth</h3>
Change `localhost` in client_secrets.json to `http://35.162.219.188`<br>
Change Authorized JavaScript origins in Google Developers Console

<h3>Configure Apache to use items_webserver.wsgi</h3>
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

<h3>Create .wsgi File </h3>
In /var/www/html/public_html/ItemCatalog/ directory:
Create items_webserver.wsgi and add
```
import sys

sys.path.append('/var/www/html/public_html/ItemCatalog/')

from items_webserver import app as application
import items_db_setup
import items_db_query
```








