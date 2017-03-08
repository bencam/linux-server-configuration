# Linux Server Configuration
This is a set of instructions on how to set up a Ubuntu Linux server to host a simple web application built with Flask.

The instructions are written specifically for hosting an app called [Nuevo M&eacute;xico](https://github.com/bencam/nuevo-mexico) on an Amazon Lightsail instance but can easily be adapted to work for an alternative application and/or server provider.


## 1. Details specific to the server I set up
The IP address is ADD.

The SSH port used is `ADD`.

The URL to the hosted webpage is simply the public IP address: ADD.


## 2. Software installed during configuration
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

`To                         Action      From
--                         ------      ----
22                         DENY        Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    DENY        Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)`

1. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Networking' tab and changing the firewall configuration to match the internal firewall settings above (only ports `80`, `123`, and `2200` should be allowed; make sure to deny the default port `22`)

1. Now, to login (on a Mac), open up the Terminal and run:

`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.

