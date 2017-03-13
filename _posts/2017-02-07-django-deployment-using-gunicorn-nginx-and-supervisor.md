---
title: Django web app deployment using Gunicorn, Supervisor and nginx
date: 2017-02-07 03:51:14 +0545
description: python, Django
categories: python
---

## Django deployment

It is easy to get started with django and hard to go on a long run and deployment is even cumbersome, if you don't follow the right principle it will kill your time.

Most of the time, for development, I use Ubuntu, for deployment, Centos has done me a favor as it's just become a defacto os for deployment.

here is my workflow.

### Setup a VPS.
- spin up your vps with minimal centos installation with required libraries.
```bash
#!/usr/bin/env bash
# Update the system
sudo yum update
# create a user that will deploy a django web applications.
useradd webapps
# Add a user to wheel group
usermod -aG wheel webapps
# Install epel repository
sudo yum install epel-release
# Install necessary libraries
sudo "yum install Development Tools" -y
# Install python related packages
sudo yum install python3 python34-devel python-virtualenv python-pip
# Install development tools
sudo yum install vim git links
# Install database packages
yum install postgresql-devel postgresql-libs postgresql postgresql-server \
postgresql-contrib -y
# Install database driver
sudo yum install python-psycopg2
# Image related libraries
sudo yum install libtiff-devel libjpeg-devel libzip-develfreetype-devel \
lcms2-devel libwebp-devel tcl-devel tk-devel -y
# some DNS utilities
yum install bind-utils
```

### Prepare postgresql database
For most of my application, I use Postgresql, so, let's install postgresql in centos distribution.
```bash
# Install database packages
yum install postgresql-devel postgresql-libs postgresql postgresql-server \
postgresql-contrib -y
# Install database driver
sudo yum install python-psycopg2
```

After installing postgresql, let's start postgresql server and enable postgresql for different run levels.

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

After successful installation of postgresql, it's time to prepare a database for our webapp.
postgresql create a user `postgres` as a super user during installation, we will use `postgres` user to create database for our webapp, a database user and a database user password.

```bash
# login using postgres user
sudo su - postgres
# create a database
createdb example_db
# create a user for example_db
createuser example_user --interactive
# login to psql shell
psql example_db
# set a password for example_user
ALTER USER example_user WITH PASSWORD 'yourpassword';
# grant all privileges for example user on example_db database
GRANT ALL ON DATABASE example_db to example_user;
# quit the database
\q
```


Now it's time to configure our webapp so that we can use a recently created database. Open your `settings.py` or any other database configuration file as per your web application architecture. Configure the following values as shown below.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'example_db',
        'USER': 'example_user',
        'PASSWORD': 'yourpassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
```

Ok, we are done with database server configuration. Now its time to install and setup `Gunicorn`.

### Gunicorn install and setup.
Issue the following commands in your `centos` VPS to install gunicorn.
```
#.
 (venv)# pip install gunicorn
```
After installing gunicorn, we will create a `gunicorn_start.sh` script to run the gunicorn server, which will ultimately controlled by `supervisorctl`.

On your project directory create a script called `gunicorn_start.sh` as shown below.
This script will used to launch gunicorn server.

```bash
#!/bin/bash
# Purpose: Gunicorn starter
# Author: manojit.gautam@gmail.com
# Name of an application
NAME="Your projectname"
# project directory
PROJECTDIR=/webapps/example.com
# django project virutalenv directory
VENVDIR=/webapps/example.com/venv
# Project source directory
SRCDIR=/webapps/example.com/master/src
# Sock file as gunicorn will communicate using unix socket
SOCKFILE=$PROJECTDIR/gunicorn.sock
# User who runs the app
USER=webapps
# the group to run as
GROUP=webapps
# how many worker processes should Gunicorn spawn
NUM_WORKERS=3
# which settings file should Django use
# If you haven't spit your file it should example.settings only
DJANGO_SETTINGS_MODULE=example.settings.production
# WSGI module name
DJANGO_WSGI_MODULE=example.wsgi
# Activate the virtual environment
source $VENVDIR/bin/activate
# Export the settings module
export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
# Export the python path from virtualenv dir
export PYTHONPATH=$DJANGODIR:$PYTHONPATH
# move to src dir !IMPORTANT otherwise it won't work.
cd $SRCDIR
# Start your Django Unicorn
# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
exec $VENVDIR/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $NUM_WORKERS \
  --user=$USER --group=$GROUP \
  --bind=unix:$SOCKFILE \
  --log-level=debug \
  --log-file=-
```
After configuring `gunicorn_start.sh` script, we need to install and configure `supervisor` to manage our webapp.

### Install and Configure supervisor
Install supevisor.
``` bash
  # install Supervisor
  sudo yum install Supervisor
  # create the supervisor configuration script
  # you need to create this script to work
  echo_supervisord_conf > /etc/supervisord.conf
  # create the site specific configuration and include in supervisord.conf file
  # to include the below use include directives
  [include]
  files = /etc/supervisor/conf.d/*.conf
```
To, manage and start `gunicorn_start.sh` web application starter script, we need to create a web application specific configuration file.

Create a configuration file `example.com.conf` inside `/etc/supervisor/conf.d/` directory. as shown below.

```bash
touch /etc/supervisor/conf.d/example.com.conf
```
Paste the following contents in `example.com.conf` file.


```bash
[program:example]
command=/webapps/example.com/gunicorn_start.sh              
user=webapps                                                          
stdout_logfile=/webapps/example.com/logs/gunicorn_supervisor.log   
redirect_stderr=true                                                
environment=LANG=en_US.UTF-8,LC_ALL=en_US.UTF-8  
```  

### Install nginx
We will use `Nginx` as a webserver for our django web application.
Issue the following commands to install nginx.

```bash
sudo yum install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```
Create a web server configuration file for Django web app inside `/etc/nginx/conf.d/` directory as `example.conf`
as shown below.

```bash
upstream example {
  # fail_timeout=0 means we always retry an upstream even if it failed
  # to return a good HTTP response (in case the Unicorn master nukes a
  # single worker for timing out).

  server unix:/webapps/example.com/gunicorn.sock fail_timeout=0;
}

server{
	listen 80;
	server_name example.com;
	client_max_body_size 4G;
	access_log /webapps/example.com/logs/nginx-access.log;
	error_log /webapps/example.com/logs/nginx-error.log;


	# Robot.txt configuration
  # developers work on robot.txt more so it is suitable to push inside source code.
      	location /robots.txt {
      	  alias /webapps/example.org/robots.txt;
	}
	# Static assets configuration
 	location /static/ {
        alias   /webapps/example.com/master/src/assets/;
        expires 30d;
    	}

	# Media configuration
	location /media/ {
        alias   /webapps/example.com/master/src/media/;
        expires 30d;
    	}

        # Need to review
      location / {
    	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   	  proxy_set_header Host $http_host;
    	proxy_redirect off;
   	  if (!-f $request_filename) {
        	proxy_pass http://example; #app name
        	break;
    		}
	}

        # Favicon configuration
        location /favicon.ico {
    	 alias /webapps/example.com/master/src/assets/img/favicon.ico;
	}

        # Prevent hidden files being serverd
	location ~ /\. { access_log off; log_not_found off; deny all; }

	# Error page configuration
        error_page 500 502 503 504 /500.html;
    	location = /500.html {
        root /webapps/example.com/master/src/static/;
   	 }
}
```

Now, restart the `nginx` server to load the configuration file for your django web application.

### Start a web application using supervisorctl

```bash
  supervisorctl start example
```

To stop the web application you can use supervisorctl as shown below.

```bash
supervisorctl stop example
```
