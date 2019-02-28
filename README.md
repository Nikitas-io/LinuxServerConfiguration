# LinuxServerConfiguration
This is my version of the Linux Server Configuration project for Udacity. This project was made on a Linux server instance, using the Amazon Lightsail service.

## The IP address and SSH port
- IP: 18.197.161.247
- SSH port: 2200

## The complete URL to the hosted web application
http://18.197.161.247

## Summary of software I installed and configuration changes I made
- After creating an Ubuntu Linux server instance, I downloaded the SSH key from Lightsail.
- I then used that key to SSH into that instance in order to complete the project's requirements.
- I updated all currently installed packages by running `sudo apt-get update` and then `sudo apt-get upgrade`. When upgrading the instance's software, I made sure to keep the local versions of the lightsail vm software so that I wouldn't break the instance.
- I changed the default SSH port from 22 to 2200 by first adding a custom TCP port in the instance's Networking tab which shows the firewall settings. I then edited the **sshd_config** file by typing:
`sudo nano /etc/ssh/sshd_config`. In the sshd_config I added the 2200 SSH port. I removed the 22 port when I was sure that the 2200 port was working.
- I also added a custom UDP port in the instance's Networking tab for NTP (port 123).
- I then configured the UFW firewall by:
Setting the default to deny all incoming traffic
`sudo ufw default deny incoming`
Setting the default to allow all outgoing traffic
`sudo ufw default allow outgoing`
Allowing ssh on port 2200
`sudo ufw allow 2200/tcp`
Allowing www (HTTP)
`sudo ufw allow www`
Allowing NTP
`sudo ufw allow ntp`
Enabling ufw
`sudo ufw enable`
- After that I rebooted the server from the Lightsail website using the reboot button.
- After logging in the instance using the 2200 ssh port, the next step was to create the *grader* user. So I ran the `sudo adduser grader` command.
- I gave the *grader* user superuser privileges by creating a new grader file in the /etc/sudoers.d directory, using the `sudo nano /etc/sudoers.d/grader` command. I filled that file with the following text: `grader ALL=(ALL:ALL) ALL`.
- After generating a new key pair (public and private keys) locally using ssh-keygen, I made sure to add the public key in the grader's *.ssh/authorized_keys* file which I created using the `mkdir` command. I named this key pair: *grader_pair*.
- I then logged-in to the instance as the *grader* using the *grader_pair* keys and gave file permissions to the .ssh directory and the authorized_keys file using the `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` respectively. 
- I configured the timezone with `sudo dpkg-reconfigure tzdata`. Chose Europe/Athens.
- Disabled ssh login for root user by setting **PermitRootLogin no** with in */etc/ssh/sshd_config*.
- Installed **apache2**, **libapache2-mod-wsgi**, **libapache2-mod-wsgi-py3** and checked if **mod_wsgi** was enabled with `sudo a2enmod wsgi`.
- I also installed **git** with `sudo apt-get install git`.
- Cloned my [catalog app project](https://github.com/Nikitas-io/ItemCatalogProject) from github into */var/www* with `git clone https://github.com/rrjoson/udacity-item-catalog.git item_catalog`.
- Started the Apache Web Server with `sudo service apache2 start`.
- Installed PostgreSQL with `sudo apt-get install postgresql` and made sure no remote connections are allowed. To be sure that this is the case just check if the contents of the */etc/postgresql/9.3/main/pg_hba.conf* file look like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5.
```
- Installed [libpq-dev](https://packages.debian.org/sid/libpq-dev) with `sudo apt-get install libpq-dev` to communicate with a PostgreSQL database backend.
- I then created a database user named **catalog**. I did this by switching to the postgres user with `sudo -u postgres -i`, running the `psql` command and then running `CREATE USER catalog WITH PASSWORD 'userpass123';`. I also altered the user's role with `ALTER USER catalog CREATEDB;` so that they can create databases.
- Then, after creating a database named **itemcatalog** with `CREATE DATABASE itemcatalog WITH OWNER catalog;`, I connected to the *itemcatalog* database as the *postgres* user with `\c itemcatalog`, changed user permissions with `REVOKE ALL ON SCHEMA public FROM public;` and `GRANT ALL ON SCHEMA public TO catalog;`, closed the connection to the database with `\q` and exited the postgres user with `exit`.
- I then changed the ownership of the */var/www/item_catalog* directory from root to grader with `sudo chown -R grader:grader item_catalog` and navigated to the *item_catalog* folder.
- Made a backup for the instance.
- In the *item_catalog* folder, I renamed my python application from **main.py** to **__init__.py** using the `mv main.py __init__.py` command. In my **__init.py** file, I made sure to change *app.run(host="0.0.0.0", port=5000)* to *app.run()*.
- I also changed the line *ENGINE = create_engine('postgresql+psycopg2://nikitas:nikitas@localhost/itemcatalog')* to *ENGINE = create_engine('postgresql+psycopg2://catalog:userpass123@localhost/itemcatalog')*, in my **__init__.py** file, my **database_setup.py** file and my **fill_database.py** file.
- Installed pip with `sudo apt-get install python-pip` to install the following dependencies:
    1. pip install flask
    2. pip install oauth2client
    3. pip install sqlalchemy
    4. pip install sqlalchemy_utils
    5. pip install requests
    6. pip install httplib2
    7. pip install datetime
    8. pip install psycopg2-binary
- Created the database schema with `sudo python database_setup.py`.
## Create a new Virtual Host
- In the */var/www/item_catalog* directory, I also created the **app.wsgi** file with `sudo nano app.wsgi`:
```
import sys
import logging
from catalog import app as application

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/item_catalog/")
application.secret_key = 'averysecretkey'
```
