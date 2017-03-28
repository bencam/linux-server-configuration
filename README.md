# Linux Server Configuration
This is a set of instructions on how to set up a Ubuntu Linux server to host a simple web application built with Flask.

The instructions are written specifically for hosting an app called [Nuevo M&eacute;xico](https://github.com/bencam/nuevo-mexico) on an Amazon Lightsail instance but can easily be adapted to work for an alternative application and/or server provider.


## 1. Details specific to the server I set up
The IP address is ADD.

The SSH port used is `ADD`.

The URL to the hosted webpage is simply the public IP address: ADD.


## 2. Software to install during the configuration
- Apache2
- mod_wsgi
- PostgreSQL
- git
- pip
- virtualenv
- httplib2
- Python Requests
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev
- Psycopg2


## 3. Configuration steps
### Create an instance with Amazon Lightsail
1. Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account

1. Follow the 'Create an instance' link

1. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

1. Choose a payment plan

1. Give the instance a unique name and click 'Create'

1. Wait for the instance to start up


### Connect to the instance on a local machine
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines (this can also be done on a Windows machine with a program such as [PuTTY](http://www.putty.org)).

1. Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

1. Click on 'Download default key'

1. A file called LightsailDefaultPrivateKey.pem will be downloaded; open this in a text editor

1. Copy the text and put it in a file called lightrail_key.rsa in the local ~/.ssh/ directory

1. Run `chmod 600 ~/.ssh/lightrail_key.rsa`

1. Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance (note that Lightsail will not allow someone to log in as `root`; `ubuntu` is the default user for Lightsail instances)


### Upgrade currently installed packages
1. Notify the system of what package updates are available by running `sudo apt-get update`

1. Download available package updates by running `sudo apt-get upgrade`


### Configure the firewall
1. Start by changing the SSH port from `22` to `2200` (open up the /etc/ssh/sshd_config file, change the port number on line 5 to `2200`, then restart SSH by running `sudo service ssh restart`; restarting SSH is a very important step!)

1. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running `sudo ufw status`

1. Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in

1. Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing

1. Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH

1. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200` so that SSH will work

1. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

1. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP

1. Run `sudo ufw deny 22` to deny port `22` (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port `2200`)

1. Run `sudo ufw enable` to enable the ufw firewall

1. Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

	```
	To                         Action      From
	--                         ------      ----
	22                         DENY        Anywhere
	2200/tcp                   ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	123/udp                    ALLOW       Anywhere
	22 (v6)                    DENY        Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	123/udp (v6)               ALLOW       Anywhere (v6)
	```

1. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed; make sure to deny the default port `22`)

1. Now, to login (on a Mac), open up the Terminal and run:

	`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.


### Create a new user named `grader`
1. Run `sudo adduser grader`

1. Enter in a new UNIX password (twice) when prompted

1. Fill out information for the new `grader` user

1. To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
1. Run `sudo visudo`

1. Search for a line that looks like this:

	`root    ALL=(ALL:ALL) ALL`

1. Add the following line below this one:

	`grader	   ALL=(ALL:ALL) ALL`

1. Save and close the visudo file

1. To verify that `grader` has sudo permissions, `su` as `grader` (run `su - grader`), enter the password, and run `sudo -l`; after entering in the password (again), a line like the following should appear, meaning `grader` has sudo permissions:

	```
	Matching Defaults entries for grader on
	    ip-XX-XX-XX-XX.ec2.internal:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

	User grader may run the following commands on
		ip-XX-XX-XX-XX.ec2.internal:
	    (ALL : ALL) ALL
	```


### Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

1. Choose a file name for the key pair (such as grader_key)

1. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

1. Log in to the virtual machine

1. Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

1. Run `touch .ssh/authorized_keys`

1. On the local machine, run `cat ~/.ssh/insert-name-of-file.pub`

1. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

1. Run `chmod 700 .ssh` on the virtual machine

1. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

1. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

1. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

Note that a pop-up window will ask for `grader`'s password.


### Configure the local timezone to UTC
1. Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)

1. Test to make sure the timezone is configured correctly by running`date`


### Install and configure Apache
1. Run `sudo apt-get install apache2` to install Apache

1. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load


### Install mod_wsgi
1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

1. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`


### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running `sudo apt-get install postgresql`

1. Open the /etc/postgresql/9.5/main/pg_hba.conf file

1. Make sure it looks like this (comments have been removed here for easier reading):

	```
	local   all             postgres                                peer
	local   all             all                                     peer
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
	```

### Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. Something like the following should appear:

	Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
	[GCC 5.4.0 20160609] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> 

### Create a new PostgreSQL user named `catalog` with limited permissions
1. PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the `postgres` user for accessing the PostgreSQL software)

1. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

1. Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

1. Next, give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

1. Finally, give the `catalog` user a password by running `\password catalog`

1. Check to make sure the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of 
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

1. Exit psql by running `\q`

1. Switch back to the `ubuntu` user by running `exit`


### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted
	- fill out information for `catalog`

1. Give the `catalog` user sudo permissions:
    
	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file
	- to verify that `catalog` has sudo permissions, `su` as `catalog` (run `sudo su - catalog`), and run `sudo -l`
	- after entering in the UNIX password, a line like the following should appear (meaning `catalog` has sudo permissions):

		```
		User catalog may run the following commands on
			ip-XX-XX-XX-XX.ec2.internal:
		    (ALL : ALL) ALL
		```

1. While logged in as `catalog`, create a database called catalog by running `createdb catalog`

1. Run `psql` and then run `\l` to see that the new database has been created

1. Switch back to the `ubuntu` user by running `exit`


### Install git and clone the catalog project
1. Run `sudo apt-get install git`

1. Create a directory called 'nuevoMexico' in the /var/www/ directory

1. Change to the 'nuevoMexico' directory, and clone the catalog project:

	`sudo git clone https://github.com/bencam/nuevo-mexico.git nuevoMexico`

	Note: the "nuevoMexico" part at the end simply changes the directory name for the repository to 'nuevoMexico' instead of the default 'nuevo-mexico'; this avoids problems later on as Apache does not like hyphens very much

1. Change the ownership of the 'nuevoMexico' directory to `ubuntu` by running (while in /var/www):

	`sudo chown -R ubuntu:ubuntu nuevoMexico/`

1. Change to the /var/www/nuevoMexico/nuevoMexico directory

1. Change the name of the application.py file to \_\_init__.py by running `mv application.py __init__.py`

1. In \_\_init__.py, find line 508:

	`app.run(host='0.0.0.0', port=8000)`

	Change this line to:

	`app.run()`

1. Open the database_setup.py file, and change the 'description' column character limit (line 46) from `250` to `850` (some of the place descriptions in the app are relatively long); to be clear, the line should look as follows (after it is changed):

	`description = Column(String(850))`

1. ADD SECTION ON OTHER database_setup.py CHANGES


### Add client_secrets.json and fb_client_secrets.json files
1. Authenticate login through Google:

	- Create a new project on the [Google API Console](console.developers.google.com)

	- Create an OAuth Client ID (under the Credentials tab), and make sure to add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript origins

	- Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login, http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect, and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs

	- Create a file called client_secrets.json in the /var/www/nuevoMexico/nuevoMexico/ directory 

	- Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

	- Add the client ID to line 16 of the templates/login.html file in the project directory

	- Add the complete file path for the client_secrets.json file in lines 33 and 63 in the \_\_init__.py file; change it from 'client_secrets.json' to '/var/www/nuevoMexico/nuevoMexico/client_secrets.json'

1. Authenitcate login through Facebook:

	- Create a new app at [Facebook for Developers](https://developers.facebook.com)

	- Make http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/ the site URL

	- Add the 'Facebook Login' product, and put http://XX.XX.XX.XX/ and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/ as the Valid OAuth redirect URIs

	- Create a file called fb_client_secrets.json file in the /var/www/nuevoMexico/nuevoMexico/ directory

	- Paste the following into the fb_client_secrets.json file:

		`{ "web": { "app_id": "INSERT_APP_ID", "app_secret": "INSERT_APP_SECRET" } }`

	- Add the Facebook App ID to line 61 of the templates/login.html file in the project directory

	- Add the complete file path for the fb_client_secrets.json file in lines 186 and 188 in the \_\_init__.py file; change it from 'fb_client_secrets.json' to '/var/www/nuevoMexico/nuevoMexico/fb_client_secrets.json'


### Set up a vitual environment and install dependencies
1. Start by installing pip (if it isn't installed already) with the following command:

	`sudo apt-get install python-pip`

1. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

1. Change to the /var/www/nuevoMexico/nuevoMexico/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running `virtualenv venv` (make sure to _not_ use `sudo` here as it can cause problems later on)

1. Activate the new environment, `venv`, by running `. venv/bin/activate`

1. With the virtual environment active, install the following dependenies (note: with the exception of the libpq-dev package, make sure to _not_ use `sudo` for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):

	`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` (Note: this will install to the global evironment)

	`pip install psycopg2`

1. In order to make sure everything was installed correctly, run `python __init__.py`; the following (among other things) should be returned:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

1. Deactivate the virtual environment by running `deactivate`


### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ called nuevoMexico.conf

1. Add the following into the file:

	```
	<VirtualHost *:80>
			ServerName XX.XX.XX.XX
			ServerAdmin ben.in.campbell@gmail.com
			WSGIScriptAlias / /var/www/nuevoMexico/nuevoMexico.wsgi
			<Directory /var/www/nuevoMexico/nuevoMexico/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/nuevoMexico/nuevoMexico/static
			<Directory /var/www/nuevoMexico/nuevoMexico/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```

	Note: the `Options -Indexes` lines ensure that listings for these directories in the browser is disabled.

1. Run `sudo a2ensite nuevoMexico` to enable the virtual host

	The following prompt will be returned:

	```
	Enabling site nuevoMexico.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

1. Run `sudo service apache2 reload`


### Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called nuevoMexico.wsgi in /var/www/nuevoMexico

1. Add the following to the file:

	```
	activate_this = '/var/www/nuevoMexico/nuevoMexico/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/nuevoMexico/")

	from nuevoMexico import app as application
	application.secret_key = '12345'
	```

1. Resart Apache: `sudo service apache2 restart`


### Switch the database in the application from SQLite to PostgreSQL
Replace line 38 in \_\_init__.py, line 70 in database_setup.py, and line 7 in populator.py with the following:

	engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')


### Disable the default Apache site
1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

1. Run `sudo service apache2 reload`  


### Change the ownership of the project direcotries
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

	sudo chown -R www-data:www-data nuevoMexico/

Note: if changes need to be made to the project files after the ownership of the directories has been switched to `www-data`, it is best to edit files as the `www-data` user; do this with the following command:

	sudo -u www-data vim INSERT_NAME_OF_FILE


### Set up the database schema and populate the database
1. While in the /var/www/nuevoMexico/nuevoMexico/ directory, activate the virtualenv by running `. venv/bin/activate`

1. Run `python database_setup.py`

1. Then run `python populator.py`

1. Resart Apache again: `sudo service apache2 restart`

1. Now open up a browser and check to make sure the app is working by going to http://XX.XX.XX.XX or http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com


## 5. A few helpful commands to know
Below is a list of commands that could be useful while setting up the server.

`lsb_release -a`<br>
Find out what version of Ubuntu is running

`whoami`<br>
Find out which user you are logged in as

`>> import flask`<br>
`>> flask.__version__`<br>
Find out which version of Flask is installed (run this within Python)

`sudo service apache2 restart`<br>
Restart Apache (use this to make sure updates are reflected on the app)

`virtualenv --version`<br>
Find out which verison of virtualenv is running

`which python`<br>
Find out where Python has been installed<br>
(Note: this is especially useful when making sure that the virtualenv is working correctly; when the virtualenv is activated and set up correctly, running `which python` should _not_ return `/usr/bin/python` but rather the file path to the directory where the virtualenv is located)

`vi + /var/log/apache2/error.log`<br>
View Apache error logs, and open the file starting with the last line

`sudo rm -rf INSERT_NAME_OF_VIRTUALENV_HERE`<br>
Delete a virtualenv and all of it's directories (be careful; this can't be undone)

`dropdb INSERT_NAME_OF_DATABASE`<br>
Drop (delete) a PostgreSQL database; this is helpful if changes are made to the database_setup.py file; the current database should always be dropped and a new one created (run this command while logged in as the `catalog` user)

`sudo apachectl stop` and `sudo apachectl start'
Stop and start Apache; stopping Apache breaks the database session, which makes it possible to drop a database


## 4. Some potentially useful information while configuring the server
### Rebooting the virtual machine
When logging in to the virtual machine, the following prompt may appear:

	*** System restart required ***

1. To restart the machine, simply run `sudo reboot`

	Note: it is important to keep in mind, that whenever the machine is rebooted, all information in RAM will disappear, the SSH session will end, and the machine will be offline for several minutes.

2. After waiting a few minutes, SSH back into the machine as normal


### Fixing `sudo: unable to resolve host ip-XX-XX-XX-XX` error
1. Open the /etc/hostname file (/etc/hostname contains the name of the machine)

1. Copy the file (it should say nothing more than `ip-10-20-29-203`, for example)

1. Paste this into the first line of the /etc/hosts file, and add the following before it:

	`localhost.localdomain`

	To be clear: the first line of the file should look like something this:

	`127.0.0.1 localhost localhost.localdomain ip-10-20-29-203`)

1. Run `sudo hostname` to make sure it worked; if it worked, something such as `ip-10-20-29-203` will be returned


### Fixing a servername error with Apache
When installing Apache, the following error may appear:

	AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message

1. To fix this, open up the /etc/apache2/apache2.conf file

1. Add in the following line at the end of the file:

	`ServerName localhost`

1. Restart Apache by running `sudo service apache2 restart`

1. If it did not work, the same error message will appear when restarting Apache

Note: this change will be overwritten when Apache is updated.


## 6. Sources
Below is a list of sources I used to complete this project.

Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

Udacity course: [Linux Command Line Basics](https://www.udacity.com/course/linux-command-line-basics--ud595)

Digital Ocean tutorial: [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

Digital Ocean tutorial: [How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)

Digital Ocean tutorial: [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

Kill the Yak [tutorial](http://killtheyak.com/use-postgresql-with-django-flask/) for using PostgreSQL with Flask or Django

The Hitchhiker’s Guide to Python [guide](http://docs.python-guide.org/en/latest/dev/virtualenvs/) on virtualenv

Flask [documentation](http://flask.pocoo.org/docs/0.12/installation/) on virtualenv

tutorialspoint [tutorial](https://www.tutorialspoint.com/postgresql/postgresql_create_database.htm) on creating a database with PostgreSQL

Digital Ocean tutorial: [An Introduction to Linux Permissions](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-permissions)

tutorialspoint [tutorial](https://www.tutorialspoint.com/postgresql/postgresql_drop_database.htm) on how to drop a PostgeSQL database

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/latest/orm/cascades.html) on cascade delete

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/rel_1_0/orm/basic_relationships.html) on one-to-many relationships

Flask [documenation](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments) on making a virtualenv work with mod_wsgi

Techarena51 [tutorial](https://techarena51.com/index.php/one-to-many-relationships-with-flask-sqlalchemy/) on one-to-many relationships with SQLAlchemy and Flask

Udacity forum posts: [firewall configuration](https://discussions.udacity.com/t/how-is-fail2ban-related-to-ufw/47638/2), logging in as [grader](https://discussions.udacity.com/t/how-to-login-to-my-aws-virtual-server-as-new-user-grader/201164/22), PostgreSQL [roles](https://discussions.udacity.com/t/postgressql-cant-alter-role-with-created-cant-create-db-at-all/198350), more on PostgreSQL [roles](https://discussions.udacity.com/t/step-10-catalog-user/157413/2), switching from SQLite to [PostgreSQL](https://discussions.udacity.com/t/how-to-move-a-flask-app-from-using-a-sqlite3-db-to-postgresql/7004/5), more on switching from SQLite to [PostgreSQL](https://discussions.udacity.com/t/sample-flask-app-500-server-error-digital-ocean-tutorial/204312/4), virtualenv [packages](https://discussions.udacity.com/t/importerror-no-module-named-psycopg2-project5/35018/2), Apache [errors](https://discussions.udacity.com/t/running-apache-problem/35719/3), [disabling](https://discussions.udacity.com/t/solved-getting-both-ip-and-amazon-ec2s-public-url-to-point-at-app/40166) the default Apache site, more on switching from SQLite to [PostgreSQL](https://discussions.udacity.com/t/completed-the-setup-and-refreshed-server-link-but-got-error/215946/19), Amazon Lightsail [URL](https://discussions.udacity.com/t/localhost-url-for-amazon-lightsail-instance/223409/3), [firewall](https://discussions.udacity.com/t/configuring-the-firewall-for-amazon-lightsail/222208/4) configuration, [virtualenv](https://discussions.udacity.com/t/installing-virtualenv/224005), [reloading](https://discussions.udacity.com/t/solved-replacing-hello-world-with-query-data-error-500-internal-server-error/215034) PostgreSQL, [populating](https://discussions.udacity.com/t/how-to-access-index-html-in-browser/157272/7) the PostgreSQL database, [create_engine](https://discussions.udacity.com/t/psycopg2-operationalerror-postgresql-error/202640) line for PostgreSQL, [client_secrets.json](https://discussions.udacity.com/t/apache-cant-find-file-in-main-directory/24498/6) error, more on [populating](https://discussions.udacity.com/t/error-with-populating-the-database/225616) the PostgreSQL database, [preventing](https://discussions.udacity.com/t/list-of-items-in-the-directory-instead-of-displaying-site/35423) the .git directory from being accessible in the browser, [more](https://discussions.udacity.com/t/disabling-directory-listings/226155) on the .git directory, Google login [issues](https://discussions.udacity.com/t/oauth-redirect-url/159158), alternate Amazon Lightsail [URL](https://discussions.udacity.com/t/project-update-amazon-lightsail-is-now-the-recommended-platform/221495/5), cascade [delete](https://discussions.udacity.com/t/cascading-delete-from-restaurant-to-its-menu-items/15066), more on cascade [delete](https://discussions.udacity.com/t/sqlite-relations-and-delete-cascade/24691/7), virtualenv and mod_wsgi and SQLAlchemy [errors](https://discussions.udacity.com/t/parent-deleted-when-child-is-deleted/229837)

Stack Overflow and related question-and-answer posts: unable to resolve host [error](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none), system restart required [issue](http://askubuntu.com/questions/258297/should-i-always-restart-the-system-when-i-see-system-restart-required), servername [error](http://askubuntu.com/questions/329323/problem-with-restarting-apache-2) with Apache, which version of [Flask](http://stackoverflow.com/questions/5285858/determining-what-version-of-flask-is-installed), installing psycopg2 by installing [libpq-dev](http://stackoverflow.com/questions/5629368/installing-psycopg2-into-virtualenv-when-postgresql-is-not-installed-on-developm), understanding Apache [errors](http://serverfault.com/questions/607873/apache-is-ok-but-what-is-this-in-error-log-mpm-preforknotice), [reload](http://serverfault.com/questions/61383/reloading-postgresql-after-configuration-changes) PostgreSQL, the difference between [reload and restart](http://askubuntu.com/questions/105200/what-is-the-difference-between-service-restart-and-service-reload), [hyphens](http://stackoverflow.com/questions/18427766/apache2-and-mod-wsgi-target-wsgi-script-cannot-be-loaded-as-python-module) and Apache, using pip with [sudo](http://stackoverflow.com/questions/15028648/is-it-acceptable-safe-to-run-pip-install-under-sudo), more on pip and [sudo](http://stackoverflow.com/questions/21055859/what-are-the-risks-of-running-sudo-pip), virtualenv and [sudo](http://stackoverflow.com/questions/19471972/how-to-avoid-permission-denied-when-using-pip-with-virtualenv), removing packages with [apt-get](http://askubuntu.com/questions/187888/what-is-the-correct-way-to-completely-remove-an-application), loggin in as [www-data](https://ubuntuforums.org/showthread.php?t=2280551), more on [www-data](http://www.linuxquestions.org/questions/linux-newbie-8/what-is-the-www-data-account-4175430741/), www-data and [chown](http://askubuntu.com/questions/124066/how-can-i-find-the-password-to-www-data-group-to-so-i-can-change-directory-acces), changing [permissions](http://superuser.com/questions/19318/how-can-i-give-write-access-of-a-folder-to-all-users-in-linux), more on [chown](http://www.linuxquestions.org/questions/linux-general-1/changing-owner-of-a-directory-recursively-284767/), git clone [options](http://stackoverflow.com/questions/8570636/change-name-of-folder-when-cloning-from-github), cascade [delete](http://stackoverflow.com/questions/5033547/sqlachemy-cascade-delete), psql [queries](http://stackoverflow.com/questions/695289/cannot-simply-use-postgresql-table-name-relation-does-not-exist), cascade [delete](http://stackoverflow.com/questions/5033547/sqlachemy-cascade-delete), SQLAlchemy [backref](http://stackoverflow.com/questions/39869793/when-do-i-need-to-use-sqlalchemy-back-populates)


