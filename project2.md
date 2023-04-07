# Lemp Stack Implementation
# *step 1: launched an ubuntu instance and installed nginx using the following commands*

![nginx_server](webserver.png)

```bash
# run an update
sudo apt update
# install nginx
sudo apt install nginx
# verify nginx is running as a service
sudo systemctl status nginx
curl http://localhost:80   #to access server locally
```
![nginx](nginx_installed.png)
![nginx](nginxup.png)

*access over web using pub IP or pub DNS*
http://<Public-IP-Address>:80
![nginx](nginx_page.png)

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

![nginx](mysql.png)

# *step 3: Installed php (this displays the content of the site to users)*
```bash
# install php and package dependencies
sudo apt install php-fpm php-mysql
php -v    # confirm php is installed by checking the version
```

![nginx](php_installed.png)

# *step 4: Configure Nginx to use PHP processor*
1. set up a domain called projectlemp in /var/wwww directory by creating a direcrtory called <projectlemp>
```bash
cd /var/www
sudo mkdir projectlemp
# assign ownership of the directory with your current system user:
sudo chown -R $USER:$USER /var/www/projectlemp
# create a new config file in Nginx’s sites-available 
cd /etc/apache2/sites-available
touch projectlemp  # this will create the file; then paste the conf below to enable nginx serve the site using /var/www/projectlemp
#/etc/nginx/sites-available/projectlemp

server {
    listen 80;
    server_name projectlemp www.projectlemp;
    root /var/www/projectlemp;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
#activate file by linking it to site-enabled directory. 
sudo ln -s /etc/nginx/sites-available/projectlemp /etc/nginx/sites-enabled/
# check for syntax error
sudo nginx -t    
# disable nginx default host
sudo unlink /etc/nginx/sites-enabled/default 
# reload nginx to apply changes
sudo systemctl reload nginx 
#create an index.html file in the document root /var/www/projectlemp
cd /var/www/projectlemp
sudo vi info.php #paste a welcome page 
"Welcome to my Home Page"
#reload browser page to see the contents using http://<Public-IP-Address>:80
```

![syntax Validation](syntax_check.png)
![home](hombe_page.png)

# *step 5: test php with Nginx*
Here, we need to create a test php file (info.php) in document root 
```bash
#create the info.php file in the document root
sudo vi /var/www/projectlemp/info.php
#add the text below, save and exit the text editor. refresh the site and you will see the php page which is the info about our server.
<?php
phpinfo();
```
![php_page](php_page.png)

```bash
# endeavor to remove the file created as it contains sensitive info
sudo rm /var/www/your_domain/info.php
```

# *step 5: Retrieve data from mysql database with php*
```bash
# connect to mysql using root account
sudo mysql -p
# create a Database
CREATE DATABASE `Tess_database`;
# create a new user and grant full privilege on the DB
CREATE USER 'Tess_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password.1';
GRANT ALL ON Tess_database.* TO 'Tess_user'@'%';
# exit mysql shell & login with new user details to test credentials 
exit
sudo mysql -u Tess_user -p
SHOW DATABASES;  
# create a test table called my_table
CREATE TABLE Tess_database.my_table (
item_id INT AUTO_INCREMENT,
content VARCHAR(255),
PRIMARY KEY(item_id)
);
# insert contents into the table
INSERT INTO Tess_database.my_table (content) VALUES ("My first item");
INSERT INTO Tess_database.my_table (content) VALUES ("My second item");
INSERT INTO Tess_database.my_table (content) VALUES ("My third item");
# confirm data was saved
SELECT * FROM Tess_database.my_table;  
exit # exit shell
```
![sqlDB](table_created.png)
![sqlTable](inserted_content.png)

# *step 6: create a php script to connect to mysql & query the content*
```bash
# create new file called my_table.php
sudo vi /var/www/projectlemp/my_table.php
# paste the below user config details to connect to DB & run queries
<?php
$user = "Tess_user";
$password = "Password.1";
$database = "Tess_database";
$table = "my_table";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>MY_TABLE</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
# you may now access the page on the browser using <ip/my_table.php>
```
![web_page](sql_contents.png)
# *voila! this concludes our lemp stack implementation.*

