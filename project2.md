# Lemp Stack Implementation
# *step 1: launched an ubuntu instance and installed nginx using the following commands*

![webserver](https://user-images.githubusercontent.com/124391569/230684079-4879ee6f-d0ae-4922-b669-aff33675cc65.png)

```bash
# run an update
sudo apt update
# install nginx
sudo apt install nginx
# verify nginx is running as a service
sudo systemctl status nginx
curl http://localhost:80   #to access server locally
```
![nginxup](https://user-images.githubusercontent.com/124391569/230684772-f453c77c-271a-4920-88a1-63a0b272a420.png)
![nginx_installed](https://user-images.githubusercontent.com/124391569/230684860-7fb8cb47-12ad-40ff-adde-aaff3d38a9e2.png)

*access over web using pub IP or pub DNS*
http://<Public-IP-Address>:80
![nginx_page](https://user-images.githubusercontent.com/124391569/230684829-240134fa-e9f7-4ac1-b8e9-a3d6c28bc38b.png)

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

![mysql](https://user-images.githubusercontent.com/124391569/230684942-ac37e446-384b-48ba-8346-221b74fe767c.png)

# *step 3: Installed php (this displays the content of the site to users)*
```bash
# install php and package dependencies
sudo apt install php-fpm php-mysql
php -v    # confirm php is installed by checking the version
```

![php_installed](https://user-images.githubusercontent.com/124391569/230684629-1d4940de-5a87-45f1-a5e5-48832f875b17.png)

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

![syntax_check](https://user-images.githubusercontent.com/124391569/230684359-0e0e0114-14e7-4eb6-9bca-dbca6a7b1a50.png)
![home_page](https://user-images.githubusercontent.com/124391569/230685064-3a14d406-de4e-483a-8518-ba67a6337e1b.png)

# *step 5: test php with Nginx*
Here, we need to create a test php file (info.php) in document root 
```bash
#create the info.php file in the document root
sudo vi /var/www/projectlemp/info.php
#add the text below, save and exit the text editor. refresh the site and you will see the php page which is the info about our server.
<?php
phpinfo();
```
![php_page](https://user-images.githubusercontent.com/124391569/230684501-5e831b74-18b9-4280-8b61-1e37749e8ba8.png)

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
![table_created](https://user-images.githubusercontent.com/124391569/230684190-d6f23c3c-89b3-4662-b43f-58f58d837e77.png)
![inserted_content](https://user-images.githubusercontent.com/124391569/230684990-7d34e06f-6a81-4f30-8a78-2a1cb9e22009.png)

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
![sql_contents](https://user-images.githubusercontent.com/124391569/230684407-c49e4710-25c5-4de3-bc71-925ec21c7d5a.png)
# *voila! this concludes our lemp stack implementation.*

