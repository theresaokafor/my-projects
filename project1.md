# Lamp Stack Implementation
# *step 1: launched an ubuntu instance and installed apache using the following commands*

image.png

```bash
# run an update
sudo apt update
# install apache
sudo apt install apache2
# verify apache is running as a service
sudo systemctl status apache2
curl http://localhost:80
# obtain ip address using the command below
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
![Screenshot 2023-03-29 144038](https://user-images.githubusercontent.com/124391569/228636500-0171aa5b-d913-4239-b9b9-03bae90ed030.png)

*access over web using pub IP or pub DNS*
http://<Public-IP-Address>:80

# *step 2: Installed mysql (the DB management system). To store and manage data for our site*
```bash
# install mysql
sudo apt install mysql-server
sudo mysql       # to login to mysql as the admin DB user: root
#set pw for root user
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
exit       #to exit the mysql shell
sudo mysql_secure_installation      # to start interactive shell script & change pw
sudo mysql -p  
exit
```

# *step 3: Installed php (this displays the content of the site to users)*
```bash
# install php and pasckage dependencies
sudo apt install php libapache2-mod-php php-mysql
php -v    # confirm php is installed by checking the version
# lamp stack is now completely installed and functional.
```

# *step 4: Created a virtual host for the website using apache*
1. set up a domain called myprojectlamp in /var/wwww directory by creating a direcrtory called <myprojectlamp>
```bash
cd /var/www
mkdir myprojectlamp
# assign ownership of the directory with your current system user:
sudo chown -R $USER:$USER /var/www/myprojectlamp
# create a new config file in Apache’s sites-available 
cd /etc/apache2/sites-available
touch myprojectlamp.conf  # this will create the conf file; then paste the conf below to enable apache serve the site using /var/www/myprojectlamp
<VirtualHost *:80>
    ServerName myprojectlamp
    ServerAlias www.myprojectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/myprojectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
ls sites-available #to see the conf file created
# enable the new virtual host
sudo a2ensite projectlamp
sudo a2dissite 000-default #disable default website present
sudo apache2ctl configtest #check for syntax error
sudo systemctl reload apache2 # reload apache
#create an index.html file in the document root /var/www/myprojectlamp
cd /var/www/myprojectlamp
sudo vi index.html #paste a welcome page 
"Welcome to my home page"
#reload browser page to see the contents using http://<Public-IP-Address>:80
```

image.png

# *step 5: enable php on the website*
Here, we need to update the order of the dir.conf file to ensure the 
```bash
sudo vi /etc/apache2/mods-enabled/dir.conf
#modify the order of the file therein to ensure index.php comes before index.html as shown below
<DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm>

sudo systemctl reload apache2
#create the index.php file in the document root
sudo vi /var/www/myprojectlamp/index.php
#add the text below, save and exit the text editor. refresh the site and you will see the php page which is the info about our server.
<?php
phpinfo();
```
![php page](https://user-images.githubusercontent.com/124391569/228635343-42101a20-0cbe-428a-8c06-f4d70b0cf8d8.png)

# *voila! this concludes our lamp stack implementation.*
