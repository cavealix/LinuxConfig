# LinuxConfig
Documentation for Udacity Fullstack Nanodegree Program Project 7

Login: ssh -i ~/.ssh/id_rsa grader@35.166.186.239 -p 2200
URL: http://ec2-35-166-186-239.us-west-2.compute.amazonaws.com/

Overview of setup
1. Download udacity_key.rsa from https://www.udacity.com/account#!/development_environment

2. Move the file to the .ssh folder on the host machine
    
    $ mv ~/Downloads/udacity_key.rsa ~/.ssh/

3. Modify the file permissions

    $ chmod 600 ~/.ssh/udacity_key.rsa

4. Connect remotely 

    $ ssh -i ~/.ssh/udacity_key.rsa root@35.166.186.239

5. Add a new user named ‘grader’

    $ sudo adduser grader

6. Still as ‘root’, grant ‘grader’ sudo permissions

    $ nano /etc/sudoers.d/

    within file, add the following and save as ‘grader’

    grader ALL=(ALL:ALL) ALL
    
    or/and?

    $ gpasswd -a grader sudo

7. On local machine create ssh key pair

    $ ssh-keygen
    (passcode optional)

    (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html)
    (https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)

8. Copy public key from local machine

    $ nano ~/.ssh/id_rsa.pub

7. Still as ‘root’, log in as ‘grader’

    $ su - grader

8. Create .ssh directory

    mkdir .ssh

9. Set permission on directory 

    $ chmod 700 .ssh

10. Create and open authorized_keys file

    $ nano .ssh/authorized_keys

11. Paste public key into authorized_keys and save

12. Set permissions on file 

    $ chmod 600 .ssh/authorized_keys

13. Login in as ‘grader’

    $ ssh -i ~/.ssh/id_rsa grader@35.166.186.239 -p 22

14. Update all currently installed packages.

    $ sudo apt-get update
    $ sudo apt-get upgrade

15. Check the local timezone is UTC.

    $ date

16. Change SSHD configs

    $ sudo nano /etc/ssh/sshd_config

    Change Port from 22 to 2200
    Change PermitRootLogin from yes/without-password to no

17. Restart SSH to instantiate changes

    $ sudo service ssh restart

18. Don’t logout of current connection without first confirming login still possible

    $ ssh -i ~/.ssh/id_rsa grader@35.166.186.239 -p 2200

19. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    $ sudo ufw default deny incoming
    $ sudo ufw default allow outgoing
    $ sudo ufw allow 2200/tcp
    $ sudo ufw allow ssh
    $ sudo ufw allow www
    $ sudo ufw allow ntp

(https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)
    
20. Enable UFW firewall

    $ sudo ufw enable



Install Apache

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps

    $ sudo apt-get install apache2

Install and Enable mod_wsgi

    $ sudo apt-get install libapache2-mod-wsgi python-dev

enable mod_wsgi

    $ sudo a2enmod wsgi

create flask app directory structure

    $ cd /var/www
    $ sudo mkdir catalog
    $ cd catalog
    $ sudo mkdir catalog
    $ cd catalog
    $ sudo nano __init__.py

In __init__.py paste:
    
    from flask import Flask
    app = Flask(__name__)
    @app.route("/")
    def hello():
        return "Hello, I love Digital Ocean!"
    if __name__ == "__main__":
        app.run()


install pip

    $ sudo apt-get install python-pip 

install virtualenv

    $ sudo pip install virtualenv

create  and run virtualenv

    $ sudo virtualenv venv
    $ source venv/bin/activate 

install flask inside venv

    (venv)$ sudo pip install Flask

run app in venv

    sudo python __init__.py

    ^c, deactivate

Configure and Enable a New Virtual Host    

    $ sudo nano /etc/apache2/sites-available/catalog.conf

paste:

    <VirtualHost *:80>
            ServerName http://ec2-35-166-186-239.us-west-2.compute.amazonaws.com
            ServerAdmin grader@http://ec2-35-166-186-239.us-west-2.compute.amazonaws.com
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

Enable the virtual host with the following command:

    $ sudo a2ensite catalog.conf
    $ sudo service apache2 restart

Create the .wsgi File

    $ cd /var/www/catalog
    $ sudo nano catalog.wsgi

paste:

    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'Add your secret key'

restart apache

    $ sudo service apache2 restart 


    Set ServerName to localhost

    $ echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf

    Enable new file

    $ sudo a2enconf fqdn

    Restart apache to instantiate changes

    $ sudo service apache2 reload


27. Setup Git and pull app from github repository

    $ cd /var/www/catalog/catalog
    $ sudo apt-get install git

    $ git config --global user.email “cavealix@gmail.com”
    $ git config --global user.name “Alix”

    Clone repository

    $ sudo git clone https://github.com/cavealix/Project5.git

    Pull all files from ‘Project5’ into /var/www/catalog/catalog

    $ sudo mv Project5/* /var/www/catalog/catalog

    Remove repository folder ‘Project5’

    $ sudo rm -r Project5

    Make Github repository inaccessible:

    Create and open .htaccess file:

    $ sudo nano /var/www/catalog/.htaccess

    Paste in:

    RedirectMatch 404 /\.git


24. Install and Config PostGRESQL

http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html

    $ sudo apt-get install postgresql



Check that no remote connections are allowed (default):

    $ sudo nano /etc/postgresql/9.3/main/pg_hba.conf

Create needed linux user for psql:

    $ sudo adduser catalog

    (password = catalog)

Change to default user postgres:

    $ sudo su - postgres

Connect to the system:

    $ psql

Add postgre user with password:

Create user with LOGIN role and set a password:

    # CREATE USER catalog WITH PASSWORD 'catalog';

Allow the user to create database tables:

    # ALTER USER catalog CREATEDB;

List current roles and their attributes: 

    # \du

Create database:

    # CREATE DATABASE catalog WITH OWNER catalog;

Connect to the database catalog 

    # \c catalog;

Revoke all rights:

    # REVOKE ALL ON SCHEMA public FROM public;

Grant only access to the catalog role:

    # GRANT ALL ON SCHEMA public TO catalog;

Exit out of PostgreSQl and the postgres user:
    
    # \q, then $ exit

Restart postgreSQL

    $ sudo service postgresql restart

Open the database setup file:

    $ sudo nano /var/www/catalog/catalog/setup_chisel_db.py

Change the line starting with "engine" to:
    
    engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

in routes.py, modules/fboauth.py and modules/goauth.py, set absolute path for ‘client_secrets.son’ and set db engine to use postgresql and restart apache2 afterwards ( sudo service apache2 restart )

    from catalog.setup_chisel_db.py import …..
    '/var/www/catalog/catalog/client_secrets.json'

install sqlalchemy and psycopg2

    $ sudo apt-get install python-psycopg2 sqlalchemy

    $ sudo pip install --upgrade oauth2client

create a copy of setup_chisel_db.py in modules folder

    $ sudo cp setup_chisel_db.py modules/setup_chisel_db.py

Create postgreSQL database schema:

    $ python setup_chisel_db.py 



25. Start Application

    $ sudo mv chisel_app.py __init__.py

    $ sudo service apache2 restart
