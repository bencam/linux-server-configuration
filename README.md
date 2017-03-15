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
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines (this can also be done on a Windows machine with a program such as PuTTY).

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

1. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Networking' tab and changing the firewall configuration to match the internal firewall settings above (only ports `80`, `123`, and `2200` should be allowed; make sure to deny the default port `22`)

1. Now, to login (on a Mac), open up the Terminal and run:

	`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.


### Create a new user named `grader`
1. Run `sudo adduser grader`

1. Enter in a new UNIX password (twice) when prompted

1. Fill out information for the new `grader` user

1. To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
1. Run `visudo`

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

1. On the local machine, run 'cat ~/.ssh/insert-name-of-file.pub`

1. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

1. Run `chmod 700 .ssh` on the virtual machine

1. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

1. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

1. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

Note that a pop-up window will ask for `grader`'s password.


### Configure the local timezone to UTC
Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)


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

	```
	Python 2.7.6 (default, Oct 26 2016, 20:30:19) 
	[GCC 4.8.4] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>>
	```

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


### Create a Linux user called catalog and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted
	- fill out information for `catalog`

1. Give the `catalog` user sudo permissions:
    
	- run `visudo`
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


### Install git and clone the catalog project
1. Run `sudo apt-get install git-all`

1. Create a directory called `nuevoMexico` in the /var/www/ directory

1. Change to the `nuevoMexico` directory, and clone the catalog project:

	`sudo git clone https://github.com/bencam/nuevo-mexico.git nuevoMexico`

	Note: the "nuevoMexico" part at the end simply changes the directory name for the repository to 'nuevoMexico' instead of the default 'nuevo-mexico'; this avoids problems later on as Apache does not like hyphens very much

1. Change the name of the application.py file to __init__.py by running `sudo mv application.py __init__.py`

1. In __init__.py, find line 508:

	`app.run(host='0.0.0.0', port=8000)`

	Change this line to:

	`app.run()`

1. Open the database_setup.py file, and change the 'description' column character limit (line 46) from `250` to `850` (some of the place descriptions in the app are relatively long); to be clear, the line should look as follows (after it is changed):

	`description = Column(String(850))`


### Add client_secrets.json and fb_client_secrets.json files
1. Authenticate login through Google:

	- Create a new project on the [Google API Console](console.developers.google.com)

	- Create an OAuth Client ID (under the Credentials tab), and make sure to add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript origins

	- Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login, http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect, and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs

	- Create a file called client_secrets.json file in the /var/www/nuevoMexico/nuevoMexico/ directory 

	- Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

	- Add the client ID to line 16 of the templates/login.html file in the project directory

	- Add the complete file path for the client_secrets.json file in lines 33 and 63 in the __init__.py file; change it from 'client_secrets.json' to '/var/www/nuevoMexico/nuevoMexico/client_secrets.json'

1. Authenitcate login through Facebook:

	- Create a new app at [Facebook for Developers](https://developers.facebook.com)

	- Make http://XX.XX.XX.XX/ the site URL

	- Add the 'Facebook Login' product, and put http://XX.XX.XX.XX/ and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/ as the Valid OAuth redirect URIs

	- Create a file called fb_client_secrets.json file in the /var/www/nuevoMexico/nuevoMexico/ directory

	- Paste the following into the fb_client_secrets.json file:

		`{ "web": { "app_id": "INSERT_APP_ID", "app_secret": "INSERT_APP_SECRET" } }`

	- Add the Facebook App ID to line 61 of the templates/login.html file in the project directory

	- Add the complete file path for the fb_client_secrets.json file in lines 185 and 187 in the __init__.py file; change it from 'fb_client_secrets.json' to '/var/www/nuevoMexico/nuevoMexico/fb_client_secrets.json'


### Set up a vitual environment and install dependencies
1. Start by installing pip (if it isn't installed already) with the following command:

	`sudo apt-get install python-pip`

1. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

1. Change the ownership and group of the project directories and files to the `www-data` user (as it is not considered a good idea for the `root` user to own these); change to the /var/www/ directory and run:

	`sudo chown -R www-data:www-data nuevoMexico/`

1. Add the `ubuntu` user as part of the `www-data` group (make sure to do this while signed in as `ubuntu` and not `www-data`):

	`sudo usermod -a -G www-data ubuntu`

1. Switch to the `www-data` user by running `sudo -s -u www-data`

1. Change to the /var/www/nuevoMexico/nuevoMexico/ directory; choose a name for a temporary environment ('catalog-venv' is used in this example), and create this environment by running `virtualenv catalog-venv` (make sure to NOT use `sudo` here as it can cause problems later on)

1. Switch back to the `ubuntu` user (run `exit`)

1. Activate the new environment, `catalog-venv` by running `. catalog-venv/bin/activate`

1. With the virtual environment active install the following dependenies:

	`sudo pip install httplib2`

	`sudo pip install requests`

	`sudo pip install --upgrade oauth2client`

	`sudo pip install sqlalchemy`

	`sudo pip install Flask-SQLAlchemy`

	`sudo pip install flask`

	`sudo apt-get install libpq-dev` (Note: this will install to the global evironment)

	`sudo pip install psycopg2`

1. In order to make sure everything was installed correctly, run `sudo python __init__.py`; the following (among other things) should be returned

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

	Note: the `Options -Indexes` lines ensure that listings for these directories in the browser has been disabled.

1. Run `sudo a2ensite nuevoMexico` to enable the virtual host


### Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called nuevoMexico.wsgi in /var/www/nuevo-mexico

1. Add the following to the file:

        ```
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/nuevoMexico/")

        from nuevoMexico import app as application
        application.secret_key = '12345'
        ```

1. Change the ownership of the nuevoMexico.wsgi file to the `www-data` user:

        `sudo chown -R www-data:www-data nuevoMexico.wsgi`

1. Resart Apache: `sudo service apache2 restart`


### Switch the database in the application from SQLite to PostgreSQL
Replace line 38 in __init__.py, line 70 in database_setup.py, and line 7 in populator.py with the following:

	`engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`


