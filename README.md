# Configuring Linux Web Servers

## i. The IP address and SSH port
IP and Port for ssh: 

## ii. The complete URL to the Web Application


## iii. A summary of software installed and configuration changes made.

* Updated & upgraded:
```
# sudo apt-get update
# sudo apt-get upgrade
# sudo apt-get autoremove
```

* Created user `grader` and give sudo privileges
```
# sudo apt-get install finger
# sudo adduser grader
# sudo touch /etc/sudoers.d/grader
```
* Edit sudoers file and
add: `grader ALL=(ALL) PASSWD:ALL`

* Changed to grader user 
```
# su grader
```
* Made a SSH key pair with `ssh-keygen` for the grader user and added it to `/home/grader/.ssh/authorized_keys`

* Configure hfw firewall:

Only allowe SSH on port 2200 on ufw as well as NTC and HTTP on standard ports
```
# sudo ufw default deny incoming
# sudo ufw default allow outgoing
# sudo ufw allow 2200/tcp
# sudo ufw allow www
# sudo ufw allow ntp
# sudo ufw logging on
# sudo ufw enable
```

* Disable the root user access:
 Do it by setting __PermitRootLogin__ to __no__ in `/etc/ssh/sshd_config`,
 whitelisted grader user by adding __AllowUsers grader__ and then restarted the service and set __Port__ to __2200__. 
 Then restarted the service.
```
$ sudo service ssh restart
```

* Set time zone to UTC
```
$ sudo ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

* Install and configure Apache to serve a Python mod_wsgi application:
```
# sudo apt-get install apache2 
# sudo apt-get install git python2.7 python-pip
# sudo apt-get install libapache2-mod-wsgi python-dev
```

* Install and configure PostgreSQL:
```
# sudo apt-get install postgresql
# sudo su - postgres
# createuser catalog -PRSd
# exit
```
* Edit pg_hba.conf for Postgres password authentication
```
# sudo nano /etc/postgresql/9.5/main/pg_hba.conf
$ sudo service postgresql restart
$ sudo adduser catalog
```
* Then configure the database for the catalog role to use, as well as system user.
```
$ sudo su - postgres
$ createdb catalog
```

* Cloned repository into apache server root 
```
# sudo git clone https://github.com/narotamsingh/item-catalog
```

* Added .htaccess file to root directory to restrict serving of the .git folder.

_.htaccess_:
```
RedirectMatch 404 /\.git
```

* Installed python extensions:
```
# sudo apt-get install python-psycopg2 python-flask python-sqlalchemy
```

* Configured app: `/etc/var/www/html/item-catalog/config.py`
```
# sudo nano config.py
--------------------
# config.py
# Configuation for the database connection

database = 'catalog'
username = 'catalog'
password = 'pwd'
client_secrets_path = '/var/www/html/item-catalog/client_secrets.json'


def connString():
    return "postgresql+psycopg2://" + username + ":" + password + "@/" + database


def clientSecrets():
    return client_secrets_path
-------------------
```
* Then and set path to application in _catalog.wsgi_:
```
import sys

sys.path.insert(0, "/var/www/html/item-catalog")

from application import app as application
```

* If need to see errors:
```
tail -60 /var/log/apache2/error.log
```

Installed pip packages:
```
$ sudo pip install bleach oauth2client requests httplib2 glances
$ sudo pip install werkzeug==0.8.3 Flask-login==0.1.3
```


* Updated client_secrets.json file to have the server host in the Google developer console setup database with 
`python db_setup.py`, and populate database with `python db_init.py`.

* Configured apache2 site conf to use the WSGI alias to point to the repo directory
and enabled mod_rewrite on apache so that the IP redirects to the domain name.
_000-default.conf_:
```
# sudo nano /etc/apache2/sites-available/000-default.conf
----------------------
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        # Include conf-available/serve-cgi-bin.conf
        WSGIScriptAlias / /var/www/html/item-catalog/catalog.wsgi
                <Directory /var/www/html/test>
                  Order allow,deny
                  Allow from all
                </Directory>
</VirtualHost>
```

* Install supporting packages
```
# sudo apt-get install python-psycopg2 python-flask
```

* Enable Apache modules and restart apache
```
$ sudo a2enmod rewrite
$ sudo a2enmod wsgi
$ sudo service apache2 restart
```
#### Post setup of extras:
```
$ sudo apt-get install glances fail2ban unattended-upgrades
$ sudo apt-get install update-notifier-common apt-listchanges
```

### Third Party Sources:
```
1.https://stackoverflow.com/questions/43657126/flask-alchemy-psycopg2-operationalerror-fatal-password-authentication-fai
2.https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting
3.https://askubuntu.com/questions/413585/postgres-password-authentication-fails
4.https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting
5.https://superuser.com/questions/179238/postgres-ident-authentication-failed
6.https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
7.https://www.a2hosting.in/kb/developer-corner/postgresql/managing-postgresql-databases-and-users-from-the-command-line
8.http://suite.opengeo.org/docs/latest/dataadmin/pgGettingStarted/firstconnect.html
9.https://www.a2hosting.in/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root
```

