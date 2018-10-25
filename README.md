# Project
This is created for completing Project 5 of the Udacity Full Stack Web Developer Course, where the specification is to record the full set of steps required to create an environment in AWS from scratch and deploy a Web Application [more specifically the [item catalog application](https://github.com/colathurv/itemcatalog.git)], that was created in Project 3. Scratch here means starting from a Ubuntu Linux Server in AWS with just root access and no other software installed.


# Application Details
The application can be accessed [here](http://ec2-52-35-194-86.us-west-2.compute.amazonaws.com/categories) for read only access. You could login to it using Google+ Signin either by clicking on the login button on it
directly using this [URL](http://ec2-52-35-194-86.us-west-2.compute.amazonaws.com/login).

# Environment Details
The IP and port of the server are 52.35.194.86 and 2200 respectively. You need to use a ssh login using RSA authentication to access the Server that will be provided as part of project submission.

# Software Installed
1. ntp
2. apache_2
3. mod_wsgi
4. postgresql
5. pip
6. virtualenv
7. flask
8. git
9. httplib2
10. requests
11. flask-seasurf
12. oauth2client
13. sqlalchemy
14. python-psycopg2
15. itemcatalog application created in project 3.
16. category_styles.css that is required for itemcatalog to render meaningfully.

#Detailed Steps
I executed this project in the following sequence in order to be modular so that every step logically builds on a previous step. Under each step  I have provided the actual command I executed with comments and where possible I have provided a partial screen output. 

## Step 1. Create the grader user and setup the user for sudo, so that all subsequent steps are performed as the sudo user.
Comments - Get the root user working by doing the following 
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ mv ~/Downloads/udacity_key.rsa ~/.ssh/

master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh -i ~/.ssh/udacity_key.rsa root@52.35.194.86
```

Comments - Login as root and add the grader user
```
root@ip-10-20-40-234:~# adduser grader
```
Comments - Change PasswordAuthentication yes to the file sshd_config going as root and restart ssh
```
root@ip-10-20-40-234:~# vi /etc/ssh/sshd_config
root@ip-10-20-40-234:~# service ssh restart
ssh stop/waiting
ssh start/running
```
Comments - Login as grader user to make sure that the login created for grader works
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh grader@52.35.194.86 -p 22
Last login: Sun Dec 20 01:07:24 2015 from 24.5.169.174
grader@ip-10-20-40-234:~$
```

Comments - Create a new file called sudoers.d, enter the following contents to it and save it.
           grader ALL=(ALL) NOPASSWD:ALL
```  
root@ip-10-20-40-234:~# touch /etc/sudoers.d/grader 
root@ip-10-20-40-234:~# vi /etc/sudoers.d/grader
```
Comments - Login as grader and check if sudo works. In order to resolve "sudo: unable to resolve host ip-10-20-40-234"
           add a second line as follows to /etc/hosts. 
```         
127.0.0.1 localhost
127.0.0.1 ip-10-20-40-234
```
Comments - Perform the following step to make sure sudo works without any errors as the grader user.
```
grader@ip-10-20-40-234:~$ vi /etc/hosts
```
## Step 2. Update and upgrade the ubuntu environment making sure it is at the latest patch level.


Comments - Install packages as the grader user by doing the following 
```
grader@ip-10-20-40-234:~$ sudo apt-get update
grader@ip-10-20-40-234:~$ sudo apt-get upgrade
```
Comments - Check packages that require a reboot bydoing the following. I found one library that needed a reboot.
```
grader@ip-10-20-40-234:~$ vi /var/run/reboot-required.pkgs
```
Comments - Reboot by doing the following
```
grader@ip-10-20-40-234:~$ sudo shutdown -r now
````
## Step 3. Override default ports and set up a uncomplicated firewall.

Comments - Change the port of sshd_config to 2200 instead of 22, restart the ssh service
           and login as the grader user as follows.
```
grader@ip-10-20-40-234:~$  sudo vi /etc/ssh/sshd_config
grader@ip-10-20-40-234:~$  sudo service ssh restart
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh grader@52.35.194.86 -p 2200
```
Comments - As grader user set up the firewall as follows-
```
grader@ip-10-20-40-234:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

grader@ip-10-20-40-234:~$ sudo ufw allow 2200/tcp
Rule added
Rule added (v6)
grader@ip-10-20-40-234:~$ sudo ufw allow 80/tcp
Rule added
Rule added (v6)
grader@ip-10-20-40-234:~$ sudo ufw allow 123/udp
Rule added
Rule added (v6)
grader@ip-10-20-40-234:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
123/udp                    ALLOW IN    Anywhere
2200/tcp (v6)              ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
123/udp (v6)               ALLOW IN    Anywhere (v6)
```
## Step 4. Setup UTC.
Comments - As grader user set up UTC as follows and in the dialog choose "None of the Above" for Geographic area and choose UTC in the next prompt.
           Do a check to make sure that you get the date fixed.
```
grader@ip-10-20-40-234:~$ sudo dpkg-reconfigure tzdata
grader@ip-10-20-40-234:/var/www/catalog/catalog$ date
Sun Dec 27 01:35:37 UTC 2015
```
## Step 5. Install ntp, apache2 and mod_wsgi.
Comments - As grader user install ntp
```
grader@ip-10-20-40-234:~$ sudo apt-get install ntp
```
Comments - As grader user install apache2
```
grader@ip-10-20-40-234:~$ sudo apt-get install apache2
```
Comments - I encountered some errors to the effect that

E: Failed to fetch http://us-west-2.ec2.archive.ubuntu.com/ubuntu/pool/main/a/ap
r/libapr1_1.5.0-1_amd64.deb  504  Gateway Time-out [IP: 52.12.240.158 80]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-mis
sing?


Comments - Ran the update  
```
grader@ip-10-20-40-234:~$ sudo apt-get update
```
Comments - I encountered the following error. 
E: Could not get lock /var/lib/apt/lists/lock - open (11: Resource temporarily 
unavailable)
E: Unable to lock directory /var/lib/apt/lists/

Comments - So I logged out and ran an update and upgrade.

```
grader@ip-10-20-40-234:~$ sudo apt-get update
grader@ip-10-20-40-234:~$ sudo apt-get upgrade
```
Comments - As grader user install mod_wsgi
```
grader@ip-10-20-40-234:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
```
Comments - I tried to start apache2 using "sudo service apache2 restart" after the above. Found that apache2 is not a recognized servic. So I installed it again.
```
grader@ip-10-20-40-234:~$ sudo apt-get install apache2
```
Comments - After the above did the following to get the correct status
```
grader@ip-10-20-40-234:~$ sudo service apache2 status
 * apache2 is running
```
I verified the above by accessing http://52.35.194.86/ and making sure I get the "Apache2 Ubuntu Default Page"

## Step 6. Install postgresql and set it up with a new user and schema.

Comments - Install postgresql
```
grader@ip-10-20-40-234:~$ sudo apt-get install postgresql postgresql-contrib
```
Comments - Create the ubuntu system user catalog 
```
grader@ip-10-20-40-234:~$ sudo adduser catalog
```
Comments - Create the postgresql database user catalog, by going as the default admin postgresql admin user 
           that comes with the postgresql distribution. Grant the CREATE and ALTER Roles to the catalog user.
```
grader@ip-10-20-40-234:~$ sudo -u postgres psql
psql (9.3.10)
Type "help" for help.

postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
CREATE ROLE
postgres=# ALTER USER catalog CREATEDB;
ALTER ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 catalog   | Create DB                                      | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}

```
Comments - Create a database called categoryitem and assign catalog as the owner of it.

```
postgres=# CREATE DATABASE categoryitem WITH owner catalog;
CREATE DATABASE
```
Comments - Connect to the database categoryitem, revoke all rights for the public user
           on the public schema. Grant all rights to public to the catalog user.
```
postgres=# \c categoryitem;
You are now connected to database "categoryitem" as user "postgres".
categoryitem=# REVOKE ALL ON SCHEMA public FROM public;
REVOKE
categoryitem=# GRANT ALL ON SCHEMA public TO catalog;
GRANT
categoryitem=# \q
```
## Step 7. Install Install python extensions to Apache and enable mod-wsgi
```
grader@ip-10-20-40-234:~$ sudo apt-get install libapache2-mod-wsgi python-dev
grader@ip-10-20-40-234:~$ sudo a2enmod wsgi
```
Enabling module wsgi.
To activate the new configuration, you need to run:
  service apache2 restart

Comments- Start apache2 
```  
grader@ip-10-20-40-234:~$ sudo service apache2 restart
 * Restarting web server apache2
AH00558: apache2: Could not reliably determine the server's fully qualified doma
in name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress 
this message
   ...done.
```
## Step 8. Install flask with any pre-reqs required apriori.

Comments - Install Python PIP as a pre-req to install Flask
```
grader@ip-10-20-40-234:~$ sudo apt-get install python-pip
```
Comments - Use PIP install virtualenv as a pre-req to install flask
```
grader@ip-10-20-40-234:~$ sudo pip install virtualenv
```
Downloading/unpacking virtualenv
  Downloading virtualenv-13.1.2-py2.py3-none-any.whl (1.7MB): 1.7MB downloaded
Installing collected packages: virtualenv
Successfully installed virtualenv
Cleaning up...


Comments - Create the directory structure for the catalog application 
```
grader@ip-10-20-40-234:/var/www$ sudo mkdir catalog
grader@ip-10-20-40-234:/var/www$ cd catalog
grader@ip-10-20-40-234:/var/www/catalog$ sudo mkdir catalog
grader@ip-10-20-40-234:/var/www$ cd catalog
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo mkdir static templates
```


Comments - Alias virtualenv
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo virtualenv venv
```
New python executable in venv/bin/python
Installing setuptools, pip, wheel...done.

Comments - Do a listing to see if the directory structure is as intended.

```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ ls 
__init__.py  static  templates  venv

grader@ip-10-20-40-234:/var/www/catalog/catalog$ ls
__init__.py  static  templates  venv
```

Comments - Give elevated permissions to virtual environment
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo chmod -R 777 venv
```

Comments - Activate venv and install flask within the virtual environment
```
(venv)grader@ip-10-20-40-234:~$ source venv/bin/activate
(venv)grader@ip-10-20-40-234:~$ pip install flask
```

Successfully built flask itsdangerous MarkupSafe
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, flask

Successfully installed Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.3 flask-0.10.1 i
tsdangerous-0.24
/home/grader/venv/local/lib/python2.7/site-packages/pip/_vendor/requests/package
s/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is
not available. This prevents urllib3 from configuring SSL appropriately and may
cause certain SSL connections to fail. For more information, see https://urllib3
.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning

## Step 9. Setup apache and make sure it is wired correctly with flask.
Comments - Edit the init file and enter a test application to see ensure that flask is setup right 
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo vi __init__.py

from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
        return "Flask Install and Setup Successful"
if __name__ == "__main__":
        app.debug = True
        app.run(host='0.0.0.0')
```
Comments - Deactivate the virtual environment
```
(venv)grader@ip-10-20-40-234:/var/www/catalog/catalog$ deactivate
grader@ip-10-20-40-234:/var/www/catalog/catalog$
```
Comments - Edit the site file below to setup the test catalog app
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo vi /etc/apache2/sites-available/catalog.conf
```
In this file enter the below snippet and save
```
<VirtualHost *>
    ServerName 52.35.194.86
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
Comments - Create a wsgi file in the catalog directory
```
grader@ip-10-20-40-234:/var/www/catalog$ sudo vi catalog.wsgi
```
In this file enter the below snippet and save
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

Comments - To get past the following nuisance error do the workaround mentioned below
           the error
AH00558: apache2: Could not reliably determine the server's fully qualified doma
in name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress th
is message
   ...done.
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo vi /etc/apache2/conf-available/fqdn.conf
```
In the fqdn.conf enter the following snippet and save 
ServerName 52.35.194.86
Comments - Enable all these configuration files
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo a2ensite fqdn
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo a2enconf catalog
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo a2enmod wsgi
```
Comments - Disable the default configuration file
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo a2ensite 000-default.conf
```
Comments - Start Apache
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo service apache2 restart
```
Comments - Test the flask app
Type http://52.35.194.86/ in a browser and make sure the following message is displayed in the browser
Flask Install and Setup Successful

## Step 10. Install git and make it web inaccessible.
```
grader@ip-10-20-40-234:~$ sudo apt-get install git
```
Setting up git-man (1:1.9.1-1ubuntu0.2) ...
Setting up git (1:1.9.1-1ubuntu0.2) ...

Comments - Set up GIT
```
grader@ip-10-20-40-234:~$ git config --global user.name "colathurv"
grader@ip-10-20-40-234:~$ git config --global user.email "colathurv@yahoo.com"
```

Comments - Make your .git inaccessible fom the web
```
grader@ip-10-20-40-234:/var/www/catalog$ sudo vi .htaccess
```
Type the following snippet and save this file
```
RedirectMatch 404 /\.git
```

## Step 11. Clone the itemcatalog application created in project3 from github.
Comments - Clone the catalog app created in Project 3 from GIT
```
grader@ip-10-20-40-234:~$ git clone https://github.com/colathurv/itemcatalog.git
```
## Step 12. Install additional software for itemcatalog application to work
Comments - Install all this software after activating venv
```
 source venv/bin/activate
 pip install httplib2
 pip install requests
 sudo pip install flask-seasurf
 sudo pip install --upgrade oauth2client
 sudo pip install sqlalchemy
 sudo apt-get install python-psycopg2
```
## Step 14. Perform any required changes to the itemcatalog application to get the READ ONLY pages working.


Comments - From the grader@ip-10-20-40-234:/var/www/catalog/catalog
           copy all the files for the itemcatalog application
```
 sudo mv ~/itemcatalog/items.csv .
 sudo mv ~/itemcatalog/category_database_setup.py .
 sudo mv ~/itemcatalog/application.py .
 sudo mv ~/itemcatalog/templates .
 ```
Comments - In the static directory create a file called category_styles.css with contents
           as in the checked in file., in the current repository.
```
grader@ip-10-20-40-234:/var/www/catalog/catalog/static$ sudo vi category_styles.css
```
Comments - Modify all occurences of 
           create_engine('sqlite:///categoryitem.db')
           by    
           engine = create_engine('postgresql://catalog:catalog@localhost/categoryitem')
           in both the following files.

category_database_setup.py
application.py

Comments - Perform the following to create and set up the DB objects in the categoryitem database.
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ python category_database_setup.py
```
This is a partial cut and paste from the screen, when I ran it -

category id =  2
categor name =  Softbound
item id = 4
item name= Introduction to Sahajmarg
item description = A simple book

item price = 1
item image = SBK0000001.jpg
item last modified = 2015-12-23 19:10:36.070133


Comments - Perform the following moves and restart apache, to make sure that the 
           catalog application can be deployed and started. 
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo mv __init__.py flask_test.py
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo mv application.py __init__.py
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo service apache2 restart
```
 * Restarting web server apache2
   ...done.

Comments - Access the browser at http://52.35.194.86/categories and this will given error that is something like
           IOError: [Errno 2] No such file or directory: 'client_secrets.json' ...

Fix it by doing the following. 

First, create a client_secrets.json file in and just use the contents used in project 3.
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo vi client_secrets.json
```
Second, in the __init__.py file, introduce an absolute path to the client secrets file in the followng 2 places.
```
APP_PATH = '/var/www/catalog/catalog/'
CLIENT_ID = json.loads(
    open(APP_PATH + 'client_secrets.json', 'r').read())['web']['client_id']

    oauth_flow = flow_from_clientsecrets(APP_PATH + 'client_secrets.json', scope='')
```
Third , make sure to set debug as False in main of __init__.py fileand and make sure there is a route for / 

Fourth, bounce apache with a  
```
sudo service apache2 restart
```
Comments - Access the browser in http://52.35.194.86/categories to make sure that the Application is coming up
           and that you can navigate through the READ ONLY pages as per project 3 functionality.
		   
## Step 15. Change IP to EC2 hostname.
Comments - Go to http://www.hcidata.info/host2ip.cgi, enter the IP address 52.35.194.86 to get the following hostname
           http://ec2-52-35-194-86.us-west-2.compute.amazonaws.com

## Step 16. Setup OAUTH to get all pages of the itemcatalog application working with login.

Comments - Go to https://console.developers.google.com/project and create new credentials. I called my new application
           to be a Web Application called "Udacity Catalog Application", by entering the following in Java Origins
           http://ec2-52-35-194-86.us-west-2.compute.amazonaws.com
           http://ec2-52-35-194-86.us-west-2.compute.amazonaws.com/outh2callback

         This step would give a client ID as well as a client secret. Download the client_secrets.json.

         Also I changed the APPLICATION_NAME in __init__.py as follows to reflect the new application associated with the new credential
         created in Google -

         APPLICATION_NAME = "Udacity Catalog Application" 

Comments - Open the client_secrets.json on the server, delete its existing contents, cut and paste the contents from client_secrets.json
```
grader@ip-10-20-40-234:/var/www/catalog/catalog$ sudo vi client_secrets.json
```

Comments - Open login.html file and insert the client id obtained from Google Console and cut and paste it in the login_html 
           as follows, within the angular brackets.
```
grader@ip-10-20-40-234:/var/www/catalog/catalog/templates$ vi login.html
```
           data-clientid="<Insert it Here>"

Comments  Bounce apache , access the application and do a round of testing. As per snapshot provided below, I created a new item item 
          that did not exist before, after logging in as myself using my Google+ account.
![project5_snapshot](https://cloud.githubusercontent.com/assets/6411605/12005608/599442ca-ab63-11e5-80ef-a962fcee8a53.GIF)


## Step 17. Create RSA Authentication for the grader user and disable password login.

Comments - Create a public-private keypair to be used with the grader user, giving an appropriate passphrase and filename
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh-keygen
```

Comments - List to see if the files have been created.
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ls -lrt | grep udacity_grader
-rw-r--r--    1 master   Administ      398 Dec 25 16:47 udacity_grader.pub
-rw-r--r--    1 master   Administ     1766 Dec 25 16:47 udacity_grader
```

Comments - Perform the followng sequence of steps to create and copy the public RSA key to the .ssh folder of the grader user.
```
grader@ip-10-20-40-234:~$ mkdir .ssh
grader@ip-10-20-40-234:~$ touch .ssh/authorized_keys
grader@ip-10-20-40-234:~$ vi .ssh_authorized_keys
            Cut and paste the contents of udacity_grader.pub here and save.
grader@ip-10-20-40-234:~$ chmod 700 .ssh
grader@ip-10-20-40-234:~$ chmod 644 .ssh/authorized_keys
```

Comments - Test the public key for the grader user.
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh -i ./udacity_grader grader@52.35.194.86 -p2200
```
Enter passphrase for key './udacity_grader':

Comments - Test the public key for the grader user
```
grader@ip-10-20-40-234:~$ sudo vi /etc/ssh/sshd_config
```

Make the Password Authentication setting to be No as follows -
```
PasswordAuthentication no
```
Comments - Restart ssh and exit.
```
grader@ip-10-20-40-234:~$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 11696
```
Comments - Test to ensure that password passed ssh login is not allowed. 
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh grader@52.35.194.86 -p2200
Permission denied (publickey).
```
## Step 18. Disable Root access.

Comments - Disable root by editing the sshd_config file, do the setting below
and restart ssh
```
grader@ip-10-20-40-234:~$ sudo vi /etc/ssh/sshd_config
PermitRootLogin no
grader@ip-10-20-40-234:~$ sudo service ssh restart
```

Comments - Check to make sure root is disabled
```
master@MASTER-PC /c/vjn/vjnspace/python/udacity/fullstack/vagrant (master)
$ ssh -i ~/.ssh/udacity_key.rsa root@52.35.194.86 -p2200
Permission denied (publickey).
```
Colathur Vijayan [VJN]
