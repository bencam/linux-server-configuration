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



