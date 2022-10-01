## LAMP STACK IMPLEMENTATION (USING AWS AS THE CLOUD PROVIDER)
### What is a technology stack? 
A technology stack is a set of technology (framework and tools) used to develop a software product. They are essential for building easy-to-maintain, scalable web applications.
A **LAMP** stack is a bundle of four different software technologies;
-Linux (the operating system), 
-Apache (the web server), 
-MySQL (the database server), and 
-PHP/Python/Perl (the progamming language)

## STEP 0 - Preparing Prerequisites
* Create an [AWS](https://aws.amazon.com) account.
* For security best practices, create an IAM user on the root account and grant it necessary administative permissions using IAM policies.
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

![EC2server](https://user-images.githubusercontent.com/114196715/193410786-5d329d24-0ee5-409c-a455-3702cc2c11ac.png)


### Connecting to the ec2 terminal
* change directory to the directory wherein your key pair is located;

` cd ~/Downloads `

* Change premissions for the private key file (.pem),

` sudo chmod 0400 <private-key-name>.pem `

* ssh into the ec2 terminal using the command prompt shell.

` ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> `

![Successful_ssh](https://user-images.githubusercontent.com/114196715/193410857-9adfca06-f04d-41f6-8c1f-54cb375cb169.png)



## STEP 1 - INSTALLING APACHE AND UPDATING THE FIREWALL
What is Apache?
Apache is an open-source web server that forms the second layer of the **LAMP** stack. The Apache module stores website files and exchanges information with a browser using HTTP, an internet protocol for transferring website information in plain text. For example, when a browser requests a webpage, the Apache HTTP server does the following:
1. Receives the request
2. Processes the request and finds the required page file
3. Sends the relevant information back to the requester/user.

* Install Apache using Ubuntu’s package manager ‘apt’:

- update a list of packages in package manager

` sudo apt update `

- run apache2 package installation

` sudo apt install apache2 `

- Verify that Apache is running

` sudo systemctl status apache2 `

* To allow our web server to receive traffic from web users, edit inbound rule on the EC2 security group. Allow HTTP traffic on port 80 from anywhere.

![EC2 inbound rule](https://user-images.githubusercontent.com/114196715/193410919-53c708c1-7789-4016-b17e-855699357c45.png)

* Check if you can access the web server locally from the ubuntu shell

```
curl http://localhost:80
or
 curl http://127.0.0.1:80
```
* To test how our Apache server will receive requests from the internet, open a web browser and access the folowing URL:

` http://<Public-IP-Address>:80 `

![EC2 web url](https://user-images.githubusercontent.com/114196715/193410986-e0251a30-05c0-4be8-aded-eee8fbefa48f.png)

* You can get the public IP of your instance to use in the above step (without navigating to the AWS console) by checking the instance metadata via the command:

` curl -s http://169.254.169.254/latest/meta-data/public-ipv4 `

## STEP 2 — installing mysql
Now that the web server is up and running, you need to install [a database management system, DBMS](https://en.wikipedia.org/wiki/Database#Database_management_system) to be able to store and manage data for your site in a relational database.
MySQL is an open-source relational database system and is the third layer of the LAMP stack. The LAMP stack model uses MySQL for storing, managing, and querying information in relational databases. For example, developers store application data, such as customer records, sales, and inventories. When a user searches for information, the web server queries the stored data in MySQL.

* Install MySQL using Ubuntu’s package manager ‘apt’:

` sudo apt install mysql-server `

- Log in to the MySQL console

` sudo mysql `

![MySQL welcome page](https://user-images.githubusercontent.com/114196715/193411058-d9d37140-d4b3-4f8b-ab67-f9071416261b.png)


* Set a password for the root user, using mysql_native_password as default authentication method. I defined the user’s password as Mukumuku.1

` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Mukumuku.1'; `

- Exit the MySQL shell with: ` mysql> exit `

* Start an interactive script that removes some insecure default settings and lock down access to your database system.

` sudo mysql_secure_installation `

- When asked if to validate password plugin, I chose "No". Enabling "validating password plugin" allows you to define password strength as either low, medium or strong. It is safe to leave this feature disabled, but always use strong, unique passwords for databases.

* Select and confirm a password for the MySQL root user (not to be confused with the system root user).
- The database root user is an administrative user with full privileges over the database system. Define a strong password here for additional security measures.
- For the rest of the questions, press Y and hit the ENTER key at each prompt.
- Test if you are able to log in to your MySQL console: ` sudo mysql -p `
- Exit the MySQL console: ` mysql> exit `

## STEP 3 — installing php
PHP(Hypertext Preprocessor) is the fourth and final layer of the LAMP stack. It is a scripting language that allows websites to run dynamic processes.
- In addition to installing the 'php package', you'll need to install 'php-mysql'(a PHP module that allows PHP to communicate with MySQL-based databases), and 'libapache2-mod-php'(to enable Apache to handle PHP files).
* Install the above listed requirements in a single run:

` sudo apt install php libapache2-mod-php php-mysql `

- After installation, confirm the version of the php: ` php -v `

To test the setup with a PHP script, it is best to set up [an Apache Virtual Host](https://httpd.apache.org/docs/2.4/vhosts/) to hold your website’s files and folders.

## STEP 4 — creating a virtual host for your website using apache
- Here, we will set up a domain called 'Projectlamp' (can be named anything)
* Create a web root directory for 'Projectlamp' using the mkdir command

` mkdir /var/www/Projectlamp `

* Assign ownership of the directory to your current system user:

` sudo chown -R $USER:$USER /var/www/Projectlamp `

* Create and open a new configuration file in Apache’s sites-available directory using vim editor;
` sudo vi /etc/apache2/sites-available/Projectlamp.conf `

- press 'i' to enter the insert mode and paste the following bare-bones configuration:

```
<VirtualHost *:80>
    ServerName Projectlamp
    ServerAlias www.Projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/Projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Press the 'esc' key, followed by :wq to save the text and exit the vim editor page.

* use the ls command to check the newly created configuration file in Apaches's sites available directory.

` sudo ls /etc/apache2/sites-available `

![ls sites-available](https://user-images.githubusercontent.com/114196715/193412242-f37f7cf4-5539-439a-89c6-82640e99abd5.png)


- As seen above, the '000-default' directory is the config file for the server block enabled by default in Apache to serve documents from the /var/www/html directory. If not disabled, it will overwrite the Projectlamp config file when one tries to access the website URL using its public IP address. 

* Enable the new virtual host using 'a2ensite' command

` sudo a2ensite Projectlamp `

* Disable apache's default website using 'a2dissite' command

` sudo a2dissite 000-default `

* To make sure your configuration file doesn’t contain syntax errors, run:

` sudo apache2ctl configtest `

* Reload Apache so these changes take effect:

` sudo systemctl reload apache2 `

* Create an index.html file in the projectlamp web directory */var/www/Projectlamp/* so that we can test that the virtual host works:

` sudo echo 'Hello MUKHTAR from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/Projectlamp/index.html `.

- Open your browser and try to access your website using the public IP address

` http://<Public-IP-Address>:80 ` 

![Final index page](https://user-images.githubusercontent.com/114196715/193412388-3d5fbb5f-7253-4c81-a864-fb4c93c3c348.png)


## STEP 5 — enable php on the website

* Edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

`sudo vim /etc/apache2/mods-enabled/dir.conf `
- I rearranged the files with index.php as the first, index.html as the second and so on:
- Original file
```
  <IfModule mod_dir.c>
        DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  </IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
- Modified
```
  <IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl: index.xhtml index.htm
  </IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
- After saving, reload the apache serve:
` sudo systemctl reload apache2 `

* Create a PHP script to test that PHP is correctly installed and configured on your server.
- Create a new file named 'index.php' in your custom web root folder
` vim /var/www/Projectlamp/index.php `
- Enter the following text:
```
<?php
phpinfo();
```
- Save the page and refresh the website URL in your browser.

![final php server](https://user-images.githubusercontent.com/114196715/193411652-304ac560-298d-437b-927b-bdf2a632fadd.png)

*  It’s best to remove the file you created as it contains sensitive information about your PHP environment -and your Ubuntu server.

`sudo rm /var/www/Projectlamp/index.php`
