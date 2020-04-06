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

sudo -u postgres psql
create database testproject;
create user testprojectuser with password 'TypePasswordHere';
grant all privileges on database testproject to testprojectuser;
\q



Creating Python Virtual Environment

cheak current Python Version 

oracle@devops:~$ python3 -V
Python 3.6.9

oracle@devops:~$ sudo apt install python3-pip
