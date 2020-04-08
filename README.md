# deploying Django,PostgreSQL and NGINX as reverse proxy for uWSGI app server on CentOS 7

freashly installed Centos 7 Virtula Machine

create non root user with sudo privilage 

<b>$useradd django </b>

set password 

<b>$passwd django</b>

As root, run this command to add your new user to the wheel group

<b>$gpasswd -a django wheel</b>

upadte the OS 

<b>$su - django</b>

<b>$sudo yum update -y</b>

we first need to enable the EPEL repository

<b>$sudo yum install epel-release</b>

check current python version 

<b>$python -V</b>
<p>2.7.5</p>

Once EPEL is enabled, we can install pip by typing:

<b>$sudo yum install python3 python3-pip </b>

# Install and Use PostgreSQL

 <b>$sudo yum install postgresql-server gcc postgresql-devel postgresql-contrib</b>
 
 Perform Initial PostgreSQL Configuration
 
<b> $sudo postgresql-setup initdb</b>
 
 <b>$sudo systemctl start postgresql</b>
<b> $sudo systemctl enable postgresql</b>
 
 With the database started, we actually need to adjust the values in one of the configuration files that has been populated.  
 start and enable PostgreSQL using systemctl
 
 <b>$sudo nano /var/lib/pgsql/data/pg_hba.conf</b>
 
 We can configure this by modifying the two host lines at the bottom of the file. Change the last column to md5. This will allow password authentication:
 
 Create a Database and Database User
 we will create two database and two user and grant privilage
 
 
<b> $sudo su - postgres</b>
 
 save and close
<b> $sudo systemctl restart postgresql</b>


 
 <b>$sudo -i -u postgres</b>
 <p> <b>$psql</b></p>
 
 
<b>postgres=#create database app1db; </b>
<b>postgres=#create user app1user with password '******';</b>
 
<b>postgres=#grant all privileges on database app1db to app1user;</b>

<b>postgres=#create database app2db;</b>
<b>postgres=#create user app2user with password '******';</b>
 
<b>postgres=#grant all privileges on database app2db to app2user;</b>



# install virtualenv and virtualenvwrapper globally by typing:

<b>$sudo pip3 install virtualenv virtualenvwrapper</b>

configure our shell with the information it needs to work with the virtualenvwrapper script. Our virtual environments will all be placed within a directory in our home folder called Env for easy access. This is configured through an environmental variable called WORKON_HOME. We can add this to our shell initialization script and can source the virtual environment wrapper script.

To add the appropriate lines to your shell initialization script, you need to run the following commands:

<p><b>$echo "export WORKON_HOME=~/Env" >> ~/.bashrc</b></p>
<p><b>$echo "source /usr/bin/virtualenvwrapper.sh" >> ~/.bashrc</b> </p>

source your shell initialization script for current session

<p><b>$source ~/.bashrc </b></p>

# Create Django Projects

<b>$ cd ~ </b>

<b>$mkvirtualenv -p python3 app1 </b>    

install Django

<b>$ pip3 install django</b>

With Django installed, we can create our first application app1:

<b>$./django-admin.py startproject app1</b>

<b>$cd app1 </b>
edit setting.py

add/update the follwwing

<b>$nano ~/app1/app1/settings.py</b>

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

<b>$./manage.py migrate</b>

 create User 

<b>$./manage.py createsuperuser </b>

collect our site’s static elements and place them within that directory by typing:

<b>$./manage.py collectstatic</b>


then test project

we can test our project by temporarily starting the development 

</b>$./manage.py runserver 0.0.0.0:8080</b>

This will start up the development server on port 8080.  to check open on browser 

http://server_domain_or_IP:8080


if successfull create another application app2

<b>$ cd ~</b>

<b>$ mkvirtualenv -p python3 app1      </b>

install Django

<b>$ $ pip3 install django </b>

With Django installed, we can create our first application app1:
<b> $django-admin.py startproject app2 </b>

<b>$cd app1 </b>
edit setting.py

add/update the follwwing

<b>$nano ~/app2/app2/settings.py</b>

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

<b>$./manage.py migrate</b>

 create User 

<b>$./manage.py createsuperuser </b>

collect our site’s static elements and place them within that directory by typing:

<b>./manage.py collectstatic</b>


then test project

we can test our project by temporarily starting the development 

</b>$./manage.py runserver 0.0.0.0:8181</b>

This will start up the development server on port 8080.  to check open on browser 

http://server_IP:8181


# Installing Nginx
<b>$sudo yum install -y nginx  </b>

create two site-avliable & site-enabled folder on nginx for proxy baypassing for the two app1 & app2 django application


<b>$sudo mkdir  /etc/nginx/sites-available</b>
<b>$suod mkdir  /etc/nginx/sites-available</b>


create two server block app1.conf & app2.conf in site avaliable 

<b>$sudo nano /etc/nginx/sites-available/app1.conf</b>
<b>$sudo nano /etc/nginx/sites-available/app2.conf</b>

add the following line on /etc/nginx/nginx.conf to include sites-enabled configuartion

include /etc/nginx/sites-enabled/*.conf;

<b>$sudo nano /etc/nginx/nginx.conf</b>


# Setting up the uWSGI Application Server

Now that we have two Django projects app1 & app2 set up and ready to go, we can configure uWSGI

install uWSGI globally

<b>$sudo pip3 install uwsgi</b>

test this application server by passing it the information for app1. 

<b>$uwsgi --http :8080 --home /home/django/Env/app1 --chdir /home/djang/app1 -w app1.wsgi</b>


# Creating Configuration Files uwsgi

Running uWSGI from the command line is useful for testing, but isn’t particularly helpful for an actual deployment. Instead, we will run uWSGI in “Emperor mode”, which allows a master process to manage separate applications automatically given a set of configuration files.
<b>
$sudo mkdir -p /etc/uwsgi/sites</b>
<b>$cd /etc/uwsgi/sites</b>

create uwsgi config for app1
<b>$sudo nano app1.ini</b>

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

<b>$sudo nano app2.ini</b>

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

<b>$sudo nano /etc/systemd/system/uwsgi.service</b>

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

<b>$sudo /etc/nginx/sites-available/app1.conf</b>

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

<b>$sudo /etc/nginx/sites-available/app2.conf</b>

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

<b>$sudo ln -s /etc/nginx/sites-available/app1.conf /etc/nginx/sites-sites-enabled/app1.conf</b>

<b>$sudo ln -s /etc/nginx/sites-available/app2.conf /etc/nginx/sites-sites-enabled/app2.conf</b>

test for configuration error

<b>$sudo nginx -t</b>

restart nginx server 

<b>$sudo systemctl restart nginx</b>



finally start and enable uwsgi service 

<b>$sudo systemctl start uwsgi</b>
<b>$sudo systemctl enable uwsgi</b>

check for status

<b>$sudo systemctl status uwsgi</b>


we configure nginx reverse proxy used port bases not name based 

so we have to open both 9090 & 9191 o firewall to allow connection 

<p><b>$ sudo firewall-cmd --permanent --zone=public --add-port=9090/tcp</b></p>
<p><b>$ sudo firewall-cmd --permanent --zone=public --add-port=9191/tcp </b></p>
<p><b>$ sudo firewall-cmd --reload</b></p>



now test both app via nginx proxy server http://<ip>:<port>
 
open in browser 

http://<ip>:9090
http://<ip>:9191
 
 so by using nginx server block and uwsgi ini configration we can serve multiple application on the same host
 
 
 





 
