# deploying Django,PostgreSQL and NGINX as reverse proxy for uWSGI app server

Ubuntu 18 freashly installed Virtula Machine

oracle@devops:~$ sudo apt update



# Installing Nginx
sudo apt install -y nginx curl

# Installing PostgreSQL
Type the following command to install postgesql database:
sudo apt install -y postgresql postgresql-contrib

# Creating Database
You can create a database and database user for your Django application like below:

<p>sudo -u postgres psql</p>
<p>create database app1db;/<p>
<p>create user app1user with password 'password';</p>
<p>grant all privileges on database app1db to app1user;</p>
\q



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

