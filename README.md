# linux_server_project
Udacity Linux server project

This project was implemented via lightsail aws. All three layers of the application have been implemented on 34.237.139.151 and SSH access can be established on port 2200. The user id "grader" has been granted SSH and SUDO access so that the grader can fully review the project. All passwords have been disabled so the user via key which will be provided to the grader via submission notes. 

The server hosts the catalog project whose URL is https://34.237.139.151/. This project requires Facebook login to access its full functionality. 

Here are the steps used to configure the server along with the 3rd party modules installed.  
# add grader use
sudo adduser grader
su grader
mkdir .ssh
cd .ssh/
    3  touch authorized_keys
add grader to sudoers
	echo "grader ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/grader 
 sudo chmod 0700 /home/grader/.ssh
 sudo chmod 0600 /home/grader/.ssh/authorized_keys

# get software available list
sudo apt-get update

# upgrade packages
sudo apt-get upgrade

# clean up
sudo apt-get autoremove

# get finger
sudo apt-get install finger


download PEM file from AWS Uduntu shell
copy pem file to sever your going to ssh from
use the ssh-keygen command to retrieve the public key for your key pair.
 ssh-keygen -y -f DKey.pem
# copy value into authorized_keys file in .ssh dir
 ssh -i "DKey.pem"  grader@34.237.139.151

#disable root logins via passwords
	PermitRootLogin no

#force all users to use keys
sudo vi /etc/ssh/sshd_config
RSAAuthentication yes
PubkeyAuthentication yes
PasswordAuthentication no

# create firewall rule in AWS Networking tab Firewall section
+ Add another - Application = Custom, Protocol = TCP, Port = 2200
+ Add another - Application = HTTPS 

#change servers ssh listeners port
edit the port in sshd_config in /etc/ssh/ folder to 2200.

#restart SSHD
sudo systemctl restart ssh

# add and enable firewall rule on server
sudo ufw disable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow https
sudo ufw enable

#download and install apache hhtp server
sudo apt-get install apache2

# begin setup of wsgi handoff
sudo apt-get install libapache2-mod-wsgi

#add wsgi handoff to apache default conf
sudo vi /etc/apache2/sites-enabled/000-default.conf

#add the following line at the end of the <VirtualHost *:80> block
WSGIScriptAlias / /var/www/html/myapp.wsgi

# restart to reload config
sudo apache2ctl restart

# install pip
sudo apt-get install software-properties-common
sudo apt-add-repository universe
sudo apt-get update
sudo apt-get install python-pip

# get db
sudo apt-get install postgresql
sudo pip install --upgrade pip
sudo pip2 install --upgrade pip

# install python packages required for catalog project
sudo pip2 install flask packaging oauth2client redis passlib flask-httpauth
sudo pip2 install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests

#update source code to inlucde all direct paths for json secrets and db
#change ownership and group of all files 
sudo chown www-data:www-data static
sudo chown www-data:www-data templates

#setup SSL for https
#enable the SSL module
sudo a2enmod ssl
#restart the Apache service for the change to be recognized.
sudo service apache2 restart
# The first step is certificate creation. For testing purposes, or for small LANs, you need to generate a private key (ca.key) with 2048 bit encryption.
sudo openssl genrsa -out ca.key 2048
#Then generate a certificate signing request (ca.csr)
sudo openssl req -nodes -new -key ca.key -out ca.csr
#generate a self-signed certificate (ca.crt) of X509 type valid for 365 keys.
sudo openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
# Create a directory to place the certificate files we have created.
sudo mkdir /etc/apache2/ssl
#copy all certificate files to the “/etc/apache2/ssl”
sudo cp ca.crt ca.key ca.csr /etc/apache2/ssl/
#Configure apache
sudo vi /etc/apache2/sites-enabled/000-default.conf
<VirtualHost *:443>
                ServerAdmin webmaster@localhost
                DocumentRoot /var/www/html
                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined
                SSLEngine on
                SSLCertificateFile /etc/apache2/ssl/ca.crt
                SSLCertificateKeyFile /etc/apache2/ssl/ca.key
                WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost>
# restart apache
sudo /etc/init.d/apache2 restart

#setup client secrets in facebook dev and update client secrets file and login html

Utilized Udacity's Full Stack Nano Degree Part 5. Deploying to Linux Servers lesson 2 and 3. 

Also used maketescheasier article to guid me thru SSL configuration to get Facebook logins SSL requirement. 
https://www.maketecheasier.com/apache-server-ssl-support/

Used riptutorials flask example for guidance on how to link python into apache using wsgi wrapper. 
https://riptutorial.com/flask/example/23234/wsgi-application-wrapper








