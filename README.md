
# Udacity_Fullstack_Server

This readme includes instructions to access the web application, plus a summary of the steps taken to configure the server.

Access
------
This web application can be accessed at the address http://52.91.45.190/ via port 2200

Configuration Summary
---------------------

**Amazon Lightsail Setup**

 - Create Server Instance
 - Download Default Private Key from Amazon
 - Tighten Permissions on file
`chmod 600 /home/stephen/PrivateKey/LightsailDefaultPrivateKey-us-east-1.pem`
 - On the Amazon Lightsail networking tab, add a new SSH port 2200

**Login to Server**

 - Login From Terminal using Private Key
`ssh ubuntu@52.91.45.190 -p 22 -i ~/PrivateKey/LightsailDefaultPrivateKey-us-east-1.pem`

 - Update and Upgrade Packages
`sudo apt-get update
    sudo apt-get upgrade`

**Configure Firewall**

 - Update the sshd_config file and change the "listen for" ports to include Port 2200 
`sudo nano /etc/ssh/sshd_config`

 - Configure the server firewall

    `sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable`

 - Reboot server to activate firewall
`sudo service ssh restart`

 - Logout and Test the New Port
	 2. `exit`
	 3. `ssh ubuntu@52.91.45.190 -p 2200 -i ~/PrivateKey/LightsailDefaultPrivateKey-us-east-1.pem`

**Create and Configure Grader account**

 - Create public/private rsa key pair on local machine
`ssh-keygen`
 - Copy the key from the new file
`ssh-keygen -y -f stephen/.ssh/ssh-key`

 - Create Account
`Sudo adduser grader`
 - Create a Sudoers file for the grader account by copying the existing file
`sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`
 - Edit the copied file, change ubuntu to grader
`sudo nano /etc/sudoers.d/grader`
 - Create directories to store graders keys
`su - grader
mkdir .ssh
touch .ssh/authorized_keys`
 - Copy the local key generated earlier and paste it in
`sudo nano .ssh/authorized_keys`
 - Update permissions for the new directory and file
`chmod 700 .ssh
chmod 644 .ssh/authorized_keys`
 - Test Log In for the grader
`ssh grader@52.91.45.190 -p 2200 -i ~/.ssh/ssh-key`

**Set local time zone to UTC**
`sudo dpkg-reconfigure tzdata`


**Install and Configure Apache**  
Documentation: 
 - https://www.linode.com/docs/web-servers/apache/apache-web-server-on-ubuntu-14-04
 - http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html

 1. Install Apache
`sudo apt-get install apache2 apache2-doc apache2-utils`

 2. Configure Apache
	 1. Install mod_wsgi
`sudo apt-get install libapache2-mod-wsgi
sudo apt-get install python-setuptools`
	 2. Create the "/var/www/UdacityFSND/UdacityFSND.wsgi" file.
	 3. Create the "/etc/apache2/sites-available/UdacityFSND.config" file
	 4. Disable the existing config `sudo a2dissite 000-default.config`
	 5. Enable the new config `sudo a2ensite UdacityFSND.config`

 3. Restart apache
`sudo service apache2 restart`

**Download and Configure Application**  
Documentation:
 - https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

 1. Install git
`sudo apt-get install git`

 2. Use git to clone Catalog project
`cd /var/www
sudo git clone https://github.com/sciortino/Udacity_Full_Stack_Catalog_App.git`

 3. Configure virtualenv
`sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate`

 4. Install packages
`sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install oauth2client
sudo pip install requests`

**PostgreSQL Setup**  
Documentation:
 - https://help.ubuntu.com/community/PostgreSQL#Basic_Server_Setup
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

 1. Install PostgreSQL
`sudo apt-get install postgresql`

 2. Establish postgres user credentials
`sudo -u postgres psql postgres
\password postgres`

 3. Create Database
`createdb catalog;`

 4. Create catalog user
`CREATE ROLE catalog;
\password catalog`

 5. Grant permissions to catalog user
`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

 6. Confirm that remote connections are disabled
`sudo cat /etc/postgresql/9.5/main/pg_hba.conf`

**Final Changes to the App**

 1. Update all of the filepaths and database references in the application. For example, this line tells SQL Alchemy where to look for the new PostgreSQL database.
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

 2. Update the OAuth files for the new IP address
	 1. Update the "javascript_origins" URL in both the client_secret.json file and the Google API dashboard with the new Amazon IP address.
	 2.  Update the Facebook URL with the new Amazon IP address.

 3. Fix issues with Facebook OAuth in app
Documentation: 
 - https://discussions.udacity.com/t/issues-with-facebook-oauth-access-token/233840

 4. Rebuild database schema in Postgres
`python /var/www/UdacityFSND/UdacityFSND/db_setup.py`

 5. Test App
`sudo python application.py`
