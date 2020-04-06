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



Creating Python Virtual Environment

check current Python Version 

python3 -V
Python 3.6.9

sudo apt install python3-pip
