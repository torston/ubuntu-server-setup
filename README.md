# Linux-Server-Configuration-UDACITY
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

- IP Address: http://dev.project.com.xxx.xxx.xxx.xxx.xip.io/
- Link: http://xxx.xxx.xxx.xxx/

NOTE: xxx.xxx.xxx.xxx - your IP Address

Warning: SERVER IS STOPPED

## Tasks
Get your server.
1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
2. Follow the instructions provided to SSH into your server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install and configure PostgreSQL:
12. Install git.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

## Instruction

## Create instance on Amazon Lightsail
1. Go to [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Create your ssh key
    ```
    ssh-keygen
    ```
3. Upload your ssh key to Amazon
4. Create instance with Ubuntu 16.04 (select your uploaded ssh key)

## Instructions for SSH access to the instance
1. Download Private Key from Amazon
2. Move the private key file into the folder `.ssh`
	```
	cd ~/.ssh
	mv ~/Downloads/your_key.rsa ~/.ssh/
	```
3. Open your terminal and type in
	```
	chmod 600 ~/.ssh/your_key.rsa
	```
4. In your terminal, type in (default user for your instance will be **ubuntu**) 
	```
	ssh -i ~/.ssh/your_key.rsa ubuntu@xxx.xxx.xxx.xxx
	```
	
## Update all currently installed packages

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
## Change the SSH port from 22 to 2200
1. Open config and then change Port 22 to Port 2200 , save & quit.
    ```
    sudo vi /etc/ssh/sshd_config
    ```
2. Reload SSH using 
    ```
    sudo service ssh restart
    ```
3. Check **PermitRootLogin** for ssh, should be **no**, if not change:
    ```
    cat /etc/ssh/sshd_config | grep PermitRootLogin # check
    sudo vi /etc/ssh/sshd_config
    # PermitRootLogin no
    sudo service ssh restart
    ```

## Configure the Uncomplicated Firewall (UFW)
1. UFW
```
    sudo ufw default deny incoming
    sudo ufw default deny incoming
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp # or simply www
	sudo ufw allow 123/udp
	sudo ufw enable
	sudo ufw status
```
2. Setup ports vie Amazon Console in **Instance -> Networing -> Firewall**
	
## Create a new user grader & sudo

1. Create user
    ```
    sudo adduser grader
    ```
2. Add sudo
    ```
    sudo vi /etc/sudoers
    touch /etc/sudoers.d/grader
    sudo vi /etc/sudoers.d/grader
    # paste: grader ALL=(ALL:ALL) ALL
    # save and exit
    ```
3. Test sudo
    ```
    su - grader
    sudo ls -la /root
    ``` 

## Set ssh login using keys
1. Generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. Deploy public key on instance

	On your instance:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vi .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. Reload SSH using `service ssh restart`
4. Now you can use ssh to login with the new user you created

	```
	ssh grader@xxx.xxx.xxx.xxx -i ~/.ssh/grader -p 2200
	```
 
## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi-py3`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install 
    ```
    sudo apt-get install postgresql postgresql-contrib`
    sudo su - postgres # default user for PostgreSQL
    psql
    ```
2. Create new db user
    ```
    CREATE USER catalog WITH PASSWORD 'password';
    ALTER USER catalog CREATEDB;
    CREATE DATABASE catalog WITH OWNER catalog;
    \c catalog
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public TO catalog;
    \q
    exit
    ```
- Make sure no remote connections to the database are allowed. 
Check if the contents of this file `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` looks like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
 
## Install git, clone and setup your Catalog App project.
1. Install Git using 
    ```
    sudo apt-get install git
    ```
2. Clone repo:
    ```
    git clone https://github.com/torston/item-catalog.git
    ```
3. Move project into directory `var/www/FlaskApp/FlaskApp`  (create new folder if needed)

## Setup project

### Install Flask and other dependencies
  - Install pip with `sudo apt-get install python3-pip`
  - Install virtualenv `sudo pip3 install virtualenv`
  - Setup it in project folder:
  ```
  sudo virtualenv venv
  source venv/bin/activate
  ```
  - Install Flask `pip3 install Flask`
  - Install other project dependencies `sudo pip3 install httplib2 oauth2client sqlalchemy psycopg2 flask-bootstrap`

### Update path of client_secrets.json file
  - Go to Google console https://console.developers.google.com/apis
  - Go to `Credentials -> OAuth  consent screen` and add `xip.io` to **Authorised domains**
  - Add `http://dev.project.com.xxx.xxx.xxx.xxx.xip.io` to **Authorised JavaScript origins**
  - Add `http://dev.project.com.xxx.xxx.xxx.xxx.xip.io/` and `http://dev.project.com.xxx.xxx.xxx.xxx.xip.io/login` to **Authorised redirect URIs**
  - Copy new client_secrets.json and replace exited file content
  
### Configure and enable a new virtual host
  - Run this: `sudo vi /etc/apache2/sites-available/FlaskApp.conf`
  - Paste this code: 
  ```                                                                                                                      
<VirtualHost *:80>
            ServerName yourdomain.com
            ServerAdmin youemail@email.com
            WSGIProcessGroup flaskapp
            WSGIDaemonProcess flaskapp threads=15 python-home=/var/www/FlaskApp/FlaskApp/ python-path=/var/www/FlaskApp/FlaskApp
            WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
            <Directory /var/www/FlaskApp/FlaskApp/>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static /var/www/FlaskApp/FlaskApp/static
            <Directory /var/www/FlaskApp/FlaskApp/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>                                                                                                       
  ```
  - Enable the virtual host 
  ```
  sudo a2dissite 000-default.conf
  sudo a2ensite FlaskApp
  ```

### Update application DB 
- Change create engine line in your `__init__.py` and `database_setup.py` to: 
    `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
- Setup DB:
```
python3 database_setup.py
python3 database_fill.py
```

### Configurate mod_wsgi
- Create wsgi file in **/var/www/FlaskApp** with name **flaskapp.wsgi** and paste:
    ```
    activate_this = '/var/www/FlaskApp/FlaskApp/venv3/bin/activate_this.py'
    with open(activate_this) as file_:
        exec(file_.read(), dict(__file__=activate_this))
    
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/FlaskApp/FlaskApp/")
    sys.path.insert(0, "/var/www/FlaskApp/")
    
    from FlaskApp import app as application
    application.secret_key = "some_key"
    ```
- Enable mod_wsgi
```sudo a2enmod wsgi```
### Final Step
- Resatart Apache ```sudo service apache2 start```
- ```curl localhost``` and check loaded
- Open in browser: http://dev.project.com.xxx.xxx.xxx.xxx.xip.io/

## Resources

- Other Udacity students
- https://stackoverflow.com
- https://askubuntu.com/
- https://www.digitalocean.com/community/tutorials/
- Youtube videos
- https://knowledge.udacity.com/
- http://leonwang.me/post/deploy-flask
