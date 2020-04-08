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

# install virtualenv and virtualenvwrapper globally by typing:

sudo pip3 install virtualenv virtualenvwrapper

configure our shell with the information it needs to work with the virtualenvwrapper script. Our virtual environments will all be placed within a directory in our home folder called Env for easy access. This is configured through an environmental variable called WORKON_HOME. We can add this to our shell initialization script and can source the virtual environment wrapper script.

To add the appropriate lines to your shell initialization script, you need to run the following commands:

echo "export WORKON_HOME=~/Env" >> ~/.bashrc
echo "source /usr/bin/virtualenvwrapper.sh" >> ~/.bashrc

source your shell initialization script for current session

source ~/.bashrc



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

