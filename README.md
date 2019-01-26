server ip address: 3.17.74.208

server HSS port: 2200

web application url: http://3.17.74.208.xip.io/

Installed software:
  * apache2 
  * libapache2-mod-wsgi 
  * PostgreSQL
  * git 
  * pip
  * virtualenv
  * flask
  * sqlalchemy 
  * finger 
  * mod_wsgi 
  * httplib2 
  * Python Requests 
  * oauth2client 
  * libpq-dev 
  * Psycopg2
  
  **Steps:**
  
  **Lightsail instance**

  create a Linux instance in Amazon lightsail website
    
   **SSH the server**
   
   1. from Amazon lghtsail console download the instance private key
    
   2. In the local machine place the private key file in .ssh folder
    
   3. ssh the server by typing the following command 
    
    
    ssh -i ~/.ssh/instance_private_key.pem@00.00.00.00
    
    
   **Update the server**
    
   using the following commands
    
    
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    
    
   **Change SSH port**
    
   1. change the ssh port number form the file /etc/ssh/sshd_config
    
   2. on Amazon lightsail website change the instance firewall settings to the following
    
    
    HTTP	TCP	80	
    Custom	UDP	123	
    Custom	TCP	2200
    
    
   3. restart ssh 
    
    
    sudo service ssh restart
    
    
   4. ssh from the new port
    
    
    ssh -i ~/.ssh/instance_private_key.pem -p 2200 ubuntu@00.00.00.00
    
    
   **Configuer and enable Ubuntu firewall**
    
    
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200/tcp
    sudo ufw allow www
    sudo ufw allow 123/udp
    sudo ufw deny 22
    sudo ufw enable
    
    
   **creare grader account**
     
    
    sudo adduser grader
    
    
   to give the grader account sudo permission 
    
   edit the sudoers file and add the line

    
    grader ALL=(ALL:ALL) ALL
    
    
    
   **Create SSH key-gen**
     
   1. on the local machine run 
    
    
    ssh-keygen
    
    
   2. copy the content of the file key.pub
    
   on the server
    
   3. cd to the grader home directory 
    
   4. create .ssh directory
    
   5. create .ssh/authorized_keys file and paste the key on it
    
   6. change the premission of .ssh folder to 700 
   
   and the authorized_key file to 644
    
   7. change the owner of the .ssh directory to the grader
    
   8. disable passwordauthentication from the file /etc/ssh/sshd_config
    
    
**Installing packages**
    
      
      sudo apt-get install apache2
      sudo apt-get install libapache2-mod-wsgi-py3
      sudo apt-get install postgresql
      
      
   **Postgresql configuration**
     
   open postgresql terminal 
     
    
    sudo su - postgres
    sql
    postgres=# CREATE USER catalog WITH PASSWORD 'catalogdb';
    CREATE ROLE postgres=# ALTER USER catalog CREATEDB;
    CREATE DATABASE catalog WITH OWNER catalog;
    catalog=# REVOKE ALL ON SCHEMA public FROM public;
    catalog=# GRANT ALL ON SCHEMA public TO catalog;
    catalog=# \q
    exit
    
    
   **Clone catalog application**
     
      
      sudo apt-get install git
      cd /var/www
      sudo mkdir catalog
      udo chown -R grader:grader catalog 
      cd catalog
      cd /var/www/catalog
      sudo git clone github-url catalog
      sudo nano catalog.wsgi

      
      
   catalog.wsgi content
      
      
      #!/usr/bin/python
      import sys
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0,"/var/www/catalog/")
     sys.path.append("/var/www/catalog/catalog")

     from catalog import app as application
     application.secret_key = 'super_secret_key'

      
      
   create the virtual environment
      
      
      sudo pip install virtualenv 
      sudo virtualenv venv 
      source venv/bin/activate 
      sudo chmod -R 777 venv
      apt-get install python-pip 
      sudo pip install flask 
      sudo pip install httplib2 oauth2client sqlalchemy psycopg2 
      udo pip install requests 
      sudo pip install --upgrade oauth2client 
      sudo apt-get install libpq-dev 
      sudo pip install sqlalchemy_utils 
      deactivate
      
      
   rename the application.py file to __init__.py
      
      
   **Configure the virtual host **
      
   on the file /etc/apache2/sites-available/catalog.conf
      
   paste the following content
      
      
      <VirtualHost *:80>
                ServerName 3.17.74.208
                ServerAdmin admin@3.17.74.208
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
      
