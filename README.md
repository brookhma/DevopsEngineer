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

