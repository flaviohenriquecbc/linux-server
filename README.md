# Linux Server

Linux Server project for Udacity Full Stack Nanodegree

## My current values of LightSail instance
Public IP: 52.4.6.31
SSH Port: 2200
Complete URL: http://52.4.6.31

### Steps of configuration

1. Create an Ubuntu machine on Amazon Lightsail (https://lightsail.aws.amazon.com)
Get the following data after configuring the instance:
* \<IP-ADDRESS> : the public IP of the instance
* \<PATH-TO-KEY-FROM-LIGHTSAIL> : The path to the amazon lightsail instance private key.
Set a pair of keys (public, private) on the server (Lightsail) and download the private key (PK). Change the access permission of the PK running the following:
```
$ chmod 400 <PATH-TO-KEY-FROM-LIGHTSAIL>
``` 

2. Follow the instructions provided to SSH into your server.

Access the machine with ssh:

```
$ ssh ubuntu@<IP-ADDRESS> -p 22 -i <PATH-TO-KEY-FROM-LIGHTSAIL>
```

### Securing the System

3. Update all currently installed packages.

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

4. Change the SSH port from 22 to 2200. Configure the Lightsail firewall to allow it.
Open the file /etc/ssh/sshd_config
```
$ sudo nano /etc/ssh/sshd_config
```
and change the following data:
```
Port 2200
PermitRootLogin no
PasswordAuthentication no
```
restart the ssh service
```
sudo service ssh restart
```

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
# close all incoming ports
$ sudo ufw default deny incoming
# open all outgoing ports
$ sudo ufw default allow outgoing
# open ssh port
$ sudo ufw allow 2200/tcp
# open http port
$ sudo ufw allow 80/tcp
# open ntp port
$ sudo ufw allow 123/udp
# turn on firewall
$ sudo ufw enable
```

Also on Lightsail, click on the tab Networking:
Add port Custom TCP 123
Add port Custom TCP 2200
Remove port SSH TCP 22

### Give grader access.

6. Create a new user account named grader.

```
$ sudo adduser grader
```

7. Give grader the permission to sudo.
Open the file
```
$ sudo nano /etc/sudoers.d/grader
```
And set the content
```
grader ALL=(ALL) NOPASSWD:ALL
```
Create the following directories:
```
$ mkdir /home/grader/.ssh
$ touch /home/grader/.shh/authorized_keys
$ chown grader /home/grader/.ssh
$ chown grader /home/grader/.ssh/authorized_keys
```

8. Create an SSH key pair for grader using the ssh-keygen tool.
Generate on your machine the keys (private and public):
```
$ ssh-keygen
```
Copy the content of the public key and paste on the remote instance on the /home/grader/.ssh/authorized_keys . Set the permissions:
```
$ chmod 700 /home/grader/.ssh
$ chmod 600 /home/grader/.ssh/authorized_keys
```

### Prepare to deploy your project.

9. Configure the local timezone to UTC.
Configure the time zone:
```
sudo dpkg-reconfigure tzdata
```
Choose the option 'None of the Above' and then select UTC.

10. Install and configure Apache to serve a Python mod_wsgi application.
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

11. Install and configure PostgreSQL:
```
$ sudo apt-get install PostgreSQL
````

* Do not allow remote connections
* Create a new database user named catalog that has limited permissions to your catalog application database.
```
$ sudo adduser catalog
$ sudo -u postgres -i
$ postgres:~$ creatuser catalog
$ postgres:~$ createdb catalog
$ postgres:~$ psql
$ postgres=# ALTER DATABASE catalog OWNER TO catalog;
$ postgres=# ALTER USER catalog WITH PASSWORD 'catalog'
$ postgres=# \q
$ postgres:~$ exit
```

12. Install git.
```
$ sudo apt-get install git
```

### Deploy the Item Catalog project.

13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
```
git clone https://github.com/flaviohenriquecbc/item-catalog-vagrant-virtualbox-sqlite.git
```

Open project.py and database_setup.py and replace the the create_engine for:
```
engine = create_engine('postgresql://catalog:xxxx@localhost:5432/catalog')
```
This connection string has the format: postgresql://username:password@host:port/database

14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!
Install the dependencies:
```
$ sudo apt-get -y install python-pip
$ sudo pip install SQLAlchemy
$ sudo pip install psycopg2
$ sudo pip install flask
$ sudo pip install oauth2client
$ sudo pip install requests
```

Modify the file /etc/apache2/sites-enabled/000-default.conf to add the following line (just before </VirtualHost>):
```
WSGIScriptAlias / /var/www/html/myapp.wsgi
```

Modify the file /var/www/html/myapp.wsgi to add the following content:
```
#!/usr/bin/python
import sys
import os
import logging
logging.basicConfig(stream=sys.stderr)
##Replace the standard out
sys.stdout = sys.stderr
sys.path.insert(0,"/home/item-catalog-vagrant-virtualbox-sqlite/")
os.chdir("/home/item-catalog-vagrant-virtualbox-sqlite/")
from project import app as application   
```

Restart the server:
```
$ sudo apache2ctl restart
```

Tadaa! You have the item catalog working on http://\<IP-ADDRESS>
