
# LINUX Server Configuration project

This is the final project of the UDACITY Nano Degree program. It includes data from the previous project "Item - Movie Catalog" and is build on Amazon Lightsail.

## Getting started /  Prerequisites

You need to have:
1. Account for Amazon Lightsail [click here](https://aws.amazon.com/lightsail/)

2. Make sure you use UBUNTU 16.04LTS
   
3. Item Catalog Project - in my case Movie Catalog [click here](https://github.com/MataBauer/Item-Movie-Catalog-UND-FWD-)
	* you can find me on GitHub as MataBauer
	* information about the catalog and how its funtions are availabla in his *ReadMe.md* on GitHub
   

## Get the server runing!:

### Server IP: `18.197.86.176`
- Once you create an account on Amazon Lightsail AWS, you will get your's server IP address. Mine is mentioned above. Entering this IP in to web browser you can connect to my web page.

## Server URL: `http://18.197.86.176.xip.io`
- When entering this URL the Google OAUth will be functioning and users will be able to CRUD movies in to DB.
- Add new request domain to Google OAuth settings in Google Developers Console [click here](https://console.developers.google.com/apis/credentials)

## SSH port: `2200`


### Configure server
## Security 
1. Run full server upgrade 
`sudo apt-get update`
`sudo apt-get upgrade`
`sudo apt-get dist-upgrade`

2. Change default SSH port 22 -> 2200
- Default port *22* has to be changed to *2200*
- Edit line `Port 22` in `/etc/ssh/sshd_config` 

3. Configure firewall - UFW
- By default deny all incomming and allow all outgoing connections
- Allow SSH (port 22), WWW (port 80), NTP (port 123) 
- Allow TCP connections on port 2200 (new ssh port)
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow ssh`
`sudo ufw allow 2200/tcp`
`sudo ufw allow www`
`sudo ufw allow ntp`
`sudo ufw enable`

- Check firewall status
`sudo ufw status numbered`
`sudo ufw status`

- *Something extra:* Deny default ssh 22 to reduce attack surface after default port change
`sudo ufw deny 22`


4. Local time zone was changed to UTC 
`sudo timedatectl set-timezone UTC`

5. Grader access 
- User "grader" was created
- sudo access is enable
- ssh access using ssh key (Key provided to UDACITY reviewers during submition in Notes!) 

## Server Components
1. Apache installation
- Install Apache HTTP server [click here](https://httpd.apache.org) for documentation.
`sudo apt-get apache2`
- Next install `mod_wsgi` for Python3
`sudo apt-get install libapache2-mod-wsgi-py3`

2. Install and configure PostgreSQL
- `sudo apt-get install postgresql postgresql-contrib`
- Disallow remote connections in `/etc/postgresql/9.5/main/pg_hba.conf`
- Create DB user *"catalog"* and assign minimal roles
```sql
create user catalog with password 'catalog';
alter role catalog with LOGIN;
alter user catalog CREATEDB;
```

- Create catalog DB
```sql
create database catalog with OWNER catalog;
```
- Set privileges on catalog DB
```sql
revoke all on schema public from public;
grant all on schema public to catalog;
```

## Deploy Application *Movie Catalog*
All file operations must be run with sudo
1. Clone the repository from GitHub [click here](https://github.com/MataBauer/Item-Movie-Catalog-UND-FWD-) into Apache www directory
`cd /var/www/`
`git clone https://github.com/MataBauer/Item-Movie-Catalog-UND-FWD-.git`

2. Disable browsing of .git directory
- Create /var/www/Catalog/.htaccess and add to it:
	RedirectMatch 404 /\.git

3. Install support packages for our web app
`sudo apt-get install python3-pip`

`sudo pip3 install flask packaging oauth2client redis passlib flask-httpauth`
`sudo pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`


4. Modify application to use PostgreSQL instead of SQLite 
- Replace old DB connection with new PostreSQL configuration
Old: `engine = create_engine('sqlite:///movies.db')`
New: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
- Do this for all listed files: `movies_db_create.py  movies_db_mapping.py  movies_db_sanity_check.py`

5. Create new WSGI application inside the cloned directory `/var/www/Catalog/app.wsgi`
- Inspired by [click here](https://www.jakowicz.com/flask-apache-wsgi/)
- Google OAUth secret key has to be set here 
```python
/var/www/Catalog/app.wsgi
import sys
sys.path.append('/var/www/Catalog')
from application import app as application
application.secret_key = 'super_secret_key'
```

6. Register WSGI with Apache 
`/etc/apache2/sites-available/000-default.conf`

```xml
<VirtualHost *:80>
        ServerName catalog
        ServerAdmin webmaster@localhost      
        #LogLevel info ssl:warn
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIDaemonProcess application user=www-data group=www-data threads=5 home=/var/www/Catalog/
        WSGIScriptAlias / /var/www/Catalog/app.wsgi
        <directory /var/www/Catalog>
                WSGIProcessGroup application
                WSGIApplicationGroup %{GLOBAL}
                WSGIScriptReloading On
                Order deny,allow
                Allow from all
        </directory>    
</VirtualHost>
```


7. Restart Apache to expose the application to the world

8. Navigate to: [click here](http://18.197.86.176.xip.io)



# THE END

Have funn. :smirk:

### Author

Martina Bauerova
