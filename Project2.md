## LEMP STACK IMPLEMENTATION (USING AWS AS THE CLOUD PROVIDER)
A **LEMP** stack is a bundle of four different software technologies;
-Linux (the operating system), 
-Nginx (the web server), 
-MySQL (the database server), and 
-PHP/Python/Perl (the progamming language)


### Launch and Connect to the ec2 terminal
- Spin up an ec2 instance and ssh into it.
- Edit the necessary inbound rule to allow http access from anywhere.

## STEP 1 – INSTALLING THE NGINX WEB SERVER

* Install Nginx using Ubuntu’s package manager ‘apt’:

- update a list of packages in package manager

` sudo apt update `

- run Nginx package installation

` sudo apt install nginx `

- Verify that Nginx is running

` sudo systemctl status nginx `

* Check if you can access the web server locally from the ubuntu shell

```
curl http://localhost:80
or
 curl http://127.0.0.1:80
```
* To test how our Nginx server will receive requests from the internet, open a web browser and access the following URL:

` http://<Public-IP-Address>:80 `

![Nginx_running](https://user-images.githubusercontent.com/114196715/193488049-587eb59f-c883-49bd-b918-7255a9d1abda.png)

* You can get the public IP of your instance to use in the above step (without navigating to the AWS console) by checking the instance metadata via the command:

` curl -s http://169.254.169.254/latest/meta-data/public-ipv4 `

## STEP 2 — installing mysql
Now that the web server is up and running, you need to install [a database management system, DBMS](https://en.wikipedia.org/wiki/Database#Database_management_system) to be able to store and manage data for your site in a relational database.
MySQL is an open-source relational database system and is the third layer of the LEMP stack. The LEMP stack model uses MySQL for storing, managing, and querying information in relational databases.

* Install MySQL using Ubuntu’s package manager ‘apt’:

` sudo apt install mysql-server `

- Log in to the MySQL console

` sudo mysql `

## STEP 3 — installing php
Now that we have Nginx installed as our web server and MySQL as the database, we can can install PHP to process code and generate dynamic content for the web server.
- In contrast to Apache which embeds the PHP interpreter in each request, Nginx requires an external program called *PHP-FPM*; PHP fastCGI process manager. This serves as its PHP interpreter to whic Nginx pass PHP requests for processing.
- We'll also install php-mysql, a module that allows PHP to communicate with MySQL-based databases.

* The two packages are installed at once;

` sudo apt install php-fpm php-mysql `


## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR
When using Nginx web server, we can create server blocks (similar to Apache virtual host) to encapsulate our configuration details and can host more than one domain on a single server.

- Here, we will set up a domain called 'projectLEMP' (can be named anything)
- On Ubuntu 22.04, Nginx has a default server block and is configured to serve documents out of a directory at /var/www/html. We'll create our new directory in '/var/www/' as well as a new configuration file in Nginx's sites-available directory.

* Create a web root directory for 'projectLEMP' using the mkdir command

` mkdir /var/www/projectLEMP `

* Assign ownership of the directory to your current system user:

` sudo chown -R $USER:$USER /var/www/projectLEMP `

* Create and open a new configuration file in Nginx's sites-available directory using vim/nano editor;

` sudo nano /etc/nginx/sites-available/projectLEMP.conf `

- Paste the following bare-bones configuration:

```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

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
```
- Save the file and exit the nano editor by pressing CTRL+X and then y and ENTER to confirm.

* use the ls command to check the newly created configuration file in Nginx's sites available directory.

` sudo ls /etc/nginx/sites-available `

![ls_sites-available](https://user-images.githubusercontent.com/114196715/193488309-a1311fa2-efc6-4c8b-85e4-4675b84f1180.png)

* Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

` sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/ `

- This will tell Nginx to use the configuration next time it is reloaded

* Test your configuration for syntax error:

` sudo nginx -t `

* Disable the default Nginx's host that was configured to listen on port 80 by unlinking it from the sites-enabled directory.

` sudo unlink /etc/nginx/sites-enabled/default `


* Reload Nginx so these changes take effect:

` sudo systemctl reload nginx `

* Create an index.html file in the projectLEMP web directory */var/www/projectLEMP/* so that we can test that the website works:

` sudo echo 'Hello MUKHTAR from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html `.

- Open your browser and try to access your website using the public IP address

` http://<Public-IP-Address>:80 ` 

![Hello Mukhtar](https://user-images.githubusercontent.com/114196715/193488447-b1bf74fa-3e82-4224-83a9-13e788e4f4b3.png)

## STEP 5 – TESTING PHP WITH NGINX
Now that the LEMP stack is fully set up, we need to test it to validate that Nginx can correctly hand .php files off to our PHP processor.

* Create a PHP script to test that PHP is correctly installed and configured on your server.
- Create a new file named 'info.php' in your custom web root folder (/ver/www/projectLEMP)
` sudo nano /var/www/projectLEMP/info.php `

- Enter the following text:

```
<?php
phpinfo();

```
- Save the page and refresh the website URL in your browser.

![php_webpage](https://user-images.githubusercontent.com/114196715/193488588-53e868c0-a89d-438d-b1d4-351dca0fd73f.png)

*  It’s best to remove the file you created as it contains sensitive information about your PHP environment -and your Ubuntu server.

`sudo rm /var/www/projectLEMP/info.php`

## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP
Here, we will create a test database with a simple "about me info" and configure access to it such that our Nginx server will be able to query data therefrom and display it.

- create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

* I created my database (Mukhtar_DB) by logging in to the mysql console: ` sudo mysql `and running the following command:

` CREATE DATABASE `Mukhtar_DB`; `

* I created a new user named Mukhtar, using mysql_native_password as default authentication method and defined the user's password:

` CREATE USER 'Mukhtar'@'%' IDENTIFIED WITH mysql_native_password BY 'Babatunde22'; `,
- NOTE: Do not expose your database credentials.

* Give this user permission over the "Mukhtar_DB" database:

` GRANT ALL ON Mukhtar_DB.* TO 'Mukhtar'@'%'; `

* Exit the MySQL console and text if the new user has proper permissions to log in to the console:

` mysql -u Mukhtar -p `

* After loggin in, confirm that you have access to the database

` SHOW DATABASES; `

![Database](https://user-images.githubusercontent.com/114196715/193488808-d3162276-8f04-4cdb-a9aa-75fadf26c466.png)

* From the MySQL console, create a test table named about_muku:

```
CREATE TABLE Mukhtar_DB.about_muku (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );

```

* Insert a few roles of content in the test table:

```
INSERT INTO Mukhtar_DB.about_muku (content) VALUES ("My Name");
INSERT INTO Mukhtar_DB.about_muku (content) VALUES ("My Aspiration");
INSERT INTO Mukhtar_DB.about_muku (content) VALUES ("My Current Program");
INSERT INTO Mukhtar_DB.about_muku (content) VALUES ("Duration of Program");

```

*To confirm that the data was sucessfully saved to your table, run; ` SELECT * FROM Mukhtar_DB.about_muku; `

![Show_Database_content](https://user-images.githubusercontent.com/114196715/193488998-4754a34a-1ef5-478c-977d-5212f109898c.png)

- Exit the MySQL console

* Create a new PHP script that will connect to MySQL and query for our content.

` sudo nano /var/www/projectLEMP/about_muku.php `
- Enter the following content and save the file.

```
<?php
$user = "Mukhtar";
$password = "Babatunde22";
$database = "Mukhtar_DB";
$table = "about_muku";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>ABOUT MUKU</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

```

* Access the content of your website by pasting the public IP in a broswer and create a path to your .php file.

` http://<Public_domain_or_IP>/about_muku.php `

![Final_Website](https://user-images.githubusercontent.com/114196715/193489121-17b6e521-22aa-4da6-93d2-c736e28f7163.png)

*THE END *
