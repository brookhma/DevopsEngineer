# deploying Django,PostgreSQL and NGINX as reverse proxy for uWSGI app server on CentOS 7

freashly installed Centos 7 Virtula Machine

create no root user with sudo privilage 

useradd django 

set password 

passwd django

As root, run this command to add your new user to the wheel group

gpasswd -a django wheel

upadte the OS 

su - django

sudo yum update -y

we first need to enable the EPEL repository

sudo yum install epel-release

check current python version 

python -V
2.7.5

Once EPEL is enabled, we can install pip by typing:

sudo yum install python3 python3-pip 

# Install and Use PostgreSQL

 $sudo yum install postgresql-server gcc postgresql-devel postgresql-contrib
 
 Perform Initial PostgreSQL Configuration
 
 $sudo postgresql-setup initdb
 
 $sudo systemctl start postgresql
 $sudo systemctl enable postgresql
 
 With the database started, we actually need to adjust the values in one of the configuration files that has been populated.  
 start and enable PostgreSQL using systemctl
 
 sudo nano /var/lib/pgsql/data/pg_hba.conf
 
 We can configure this by modifying the two host lines at the bottom of the file. Change the last column to md5. This will allow password authentication:
 
 Create a Database and Database User
 we will create two database and two user and grant privilage
 
 
 $sudo su - postgres
 
 save and close
 $sudo systemctl restart postgresql


 
 $sudo -i -u postgres
 $psql
 
 
create database app1db;
create user app1user with password '******';
 
grant all privileges on database app1db to app1user;

create database app2db;
create user app2user with password '******';
 
grant all privileges on database app2db to app2user;



# install virtualenv and virtualenvwrapper globally by typing:

sudo pip3 install virtualenv virtualenvwrapper

configure our shell with the information it needs to work with the virtualenvwrapper script. Our virtual environments will all be placed within a directory in our home folder called Env for easy access. This is configured through an environmental variable called WORKON_HOME. We can add this to our shell initialization script and can source the virtual environment wrapper script.

To add the appropriate lines to your shell initialization script, you need to run the following commands:

echo "export WORKON_HOME=~/Env" >> ~/.bashrc
echo "source /usr/bin/virtualenvwrapper.sh" >> ~/.bashrc

source your shell initialization script for current session

source ~/.bashrc

# Create Django Projects

$ cd ~

mkvirtualenv -p python3 app1     //use -p pytho3 to select python 3 not the default python 2.7.5

install Django

$ pip3 install django

With Django installed, we can create our first application app1:

django-admin.py startproject app1

$cd app1 
edit setting.py

add/update the follwwing

$nano ~/app1/app1/settings.py

ALLOWED_HOSTS = ['*']

<p> DATABASES = { </p>
   </p> 'default': { </p>
       <p> 'ENGINE': 'django.db.backends.postgresql_psycopg2',</p>
       <p> 'NAME': 'app1db',</p>
       </p> 'USER': 'app1user',</p>
       <p> 'PASSWORD': 'TypePasswordHere',</p>
       <p> 'HOST': 'localhost',</p>
       <p> 'PORT': '',</p>
   <p> }</p>
<p>}</p>

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

$./manage.py migrate

 create User 

./manage.py createsuperuser 

collect our site’s static elements and place them within that directory by typing:

./manage.py collectstatic


then test project

we can test our project by temporarily starting the development 

./manage.py runserver 0.0.0.0:8080

This will start up the development server on port 8080.  to check open on browser 

http://server_domain_or_IP:8080


if successfull create another application app2

$ cd ~

mkvirtualenv -p python3 app1     //use -p pytho3 to select python 3 not the default python 2.7.5

install Django

$ pip3 install django

With Django installed, we can create our first application app1:

django-admin.py startproject app2

$cd app1 
edit setting.py

add/update the follwwing

$nano ~/app2/app2/settings.py

ALLOWED_HOSTS = ['*']

<p> DATABASES = { </p>
   </p> 'default': { </p>
       <p> 'ENGINE': 'django.db.backends.postgresql_psycopg2',</p>
       <p> 'NAME': 'app2db',</p>
       </p> 'USER': 'app2user',</p>
       <p> 'PASSWORD': 'TypePasswordHere',</p>
       <p> 'HOST': 'localhost',</p>
       <p> 'PORT': '',</p>
   <p> }</p>
<p>}</p>

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

$./manage.py migrate

 create User 

./manage.py createsuperuser 

collect our site’s static elements and place them within that directory by typing:

./manage.py collectstatic


then test project

we can test our project by temporarily starting the development 

./manage.py runserver 0.0.0.0:8181

This will start up the development server on port 8080.  to check open on browser 

http://server_IP:8181


# Installing Nginx
sudo apt install -y nginx  

create two site-avliable & site-enabled folder on nginx for proxy baypassing for the two app1 & app2 django application


$sudo mkdir  /etc/nginx/sites-available
$suod mkdir  /etc/nginx/sites-available


create two server block app1.conf & app2.conf in site avaliable 

$sudo nano /etc/nginx/sites-available/app1.conf
$sudo nano /etc/nginx/sites-available/app2.conf

add the following line on /etc/nginx/nginx.conf to include sites-enabled configuartion

include /etc/nginx/sites-enabled/*.conf;

$sudo nano /etc/nginx/nginx.conf


# Setting up the uWSGI Application Server

Now that we have two Django projects app1 & app2 set up and ready to go, we can configure uWSGI

install uWSGI globally

$sudo pip3 install uwsgi

test this application server by passing it the information for app1. 

uwsgi --http :8080 --home /home/django/Env/app1 --chdir /home/djang/app1 -w app1.wsgi


# Creating Configuration Files uwsgi

Running uWSGI from the command line is useful for testing, but isn’t particularly helpful for an actual deployment. Instead, we will run uWSGI in “Emperor mode”, which allows a master process to manage separate applications automatically given a set of configuration files.

$sudo mkdir -p /etc/uwsgi/sites
$cd /etc/uwsgi/sites

create uwsgi config for app1
$sudo nano app1.ini

and past the following 


http-socket = 0.0.0.0:8080
project = app1
username = django
base = /home/%(username)


chdir = %(base)/%(project)
home = %(base)/Env/%(project)
module = %(project).wsgi:application

master = true
processes = 5

uid = %(username)
socket = /run/uwsgi/%(project).sock
chown-socket = %(username):nginx
chmod-socket = 666
vacuum = true



create uwsgi config for app1

$sudo nano app2.ini

and past the following 


http-socket = 0.0.0.0:8181
project = app1
username = django
base = /home/%(username)


chdir = %(base)/%(project)
home = %(base)/Env/%(project)
module = %(project).wsgi:application

master = true
processes = 5

uid = %(username)
socket = /run/uwsgi/%(project).sock
chown-socket = %(username):nginx
chmod-socket = 666
vacuum = true


When you are finished with this, save and close the file.


# Create a Systemd Unit File for uWSGI

We now have the configuration files we need to serve our Django projects app1 & app2, but we still haven’t automated the process. Next, we’ll create a Systemd unit file to automatically start uWSGI at boot.

$sudo nano /etc/systemd/system/uwsgi.service

and paste the following

 [Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/usr/bin/bash -c 'mkdir -p /run/uwsgi; chown django:nginx /run/uwsgi'
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target

When you are finished with this, save and close the file.




now update the nginx server blocks for each app1 & app2 

$sudo /etc/nginx/sites-available/app1.conf

add the follwing

server {
    listen 0.0.0.0:9090;
    #server_name app1.com www.app1.com;

    location = favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/django/app1;
    }

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/app1.sock;
    }
}

$sudo /etc/nginx/sites-available/app2.conf

add the follwing

server {
    listen 0.0.0.0:9191;
    #server_name app2.com www.app2.com;

    location = favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/django/app2;
    }

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/app2.sock;
    }
}

now create symbolic link of both app1.conf and app2.conf  to site-enabled

sudo ln -s /etc/nginx/sites-available/app1.conf /etc/nginx/sites-sites-enabled/app1.conf
sudo ln -s /etc/nginx/sites-available/app2.conf /etc/nginx/sites-sites-enabled/app2.conf

test for configuration error

$sudo nginx -t

restart nginx server 

$sudo systemctl restart nginx



finally start and enable uwsgi service 

$sudo systemctl start uwsgi
$sudo systemctl enable uwsgi

check for status

$sudo systemctl status uwsgi





# Creating Python Virtual Environment

check current Python Version 

python3 -V
 
sudo apt install python3-pip

upgrade pip Version 

sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv



mkdir ~/project
cd ~/project

virtualenv projectenv


source project/bin/activate
pip install django gunicorn psycopg2-binary


know we have all of the required software in place needed to start a Django project.

# Creating New Django Project
 

django-admin.py startproject app1 ~/project

# Adjust the Project Settings
adjust the settings parameters. Open the settings file in text editor and add, update and replace the following parameters

nano ~/project/app1/settings.py

ALLOWED_HOSTS = ['0.0.0.0', 'localhost']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'app1db',
        'USER': 'app1user',
        'PASSWORD': 'p@ssw0rd',
        'HOST': 'localhost',
        'PORT': '',
    }
}

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')


Create an administrative user for the project by typing:

~/project/manage.py createsuperuser

You will have to select a username, provide an email address, and choose and confirm a password.

You can collect all of the static content into the directory location you configured by typing:

~/project/manage.py collectstatic

Output
119 static files copied to '/home/oracle/project/static'.


If UFW firewall protecting your server, you'll have to allow access to the port 

sudo ufw allow 8080

test your project by starting up the Django development server with the following command:

~/project/manage.py runserver 0.0.0.0:8080


Open up your web browser, access your server's name or IP address followed by port like:

http://<ip-address>:8080
 
#  Testing Gunicorn
test Gunicorn functionality to make sure that it can serve the application.  

cd ~/project
gunicorn --bind 0.0.0.0:8080 app1.wsgi

# Creating Systemd Socket and Service Files for Gunicorn

sudo nano /etc/systemd/system/gunicorn.socket


Inside, we will create a [Unit] section to describe the socket, a [Socket] section to define the socket location, and an [Install] section to make sure the socket is created at the right time:

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

Save and close the file when you are done.


Next, create and open a systemd service file for Gunicorn with sudo privileges in your text editor. The service filename should match the socket filename with the exception of the extension:

sudo nano /etc/systemd/system/app1.service

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=peter
Group=www-data
WorkingDirectory=/home/oracle/project
ExecStart=/home/oracle/project/projectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          app1.wsgi:application

[Install]
WantedBy=multi-user.target

Save and close it now.


You can start and enable the Gunicorn socket. This will create the socket file at /run/gunicorn.sock now and at boot. When a connection is made to that socket, systemd will automatically start the gunicorn.service to handle it:

sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

You can confirm that the operation was successful by checking for the socket file.

sudo systemctl status gunicorn.socket


Next, check for the existence of the gunicorn.sock file within the /run directory:

file /run/gunicorn.sock

# Testing Socket Activation

Currently, if you've only started the gunicorn.socket unit, the gunicorn.service will not be active yet since the socket has not yet received any connections. You can check this by typing:
sudo systemctl status gunicorn

To test the socket activation mechanism, you can send a connection to the socket through curl by typing:

curl --unix-socket /run/gunicorn.sock localhost


# Configure Nginx to Proxy Pass to Gunicorn

Start by creating and opening a new server block in Nginx's sites-available directory:

sudo nano /etc/nginx/sites-available/testproject.conf

