## CLIENT-SERVER ARCHITECTURE WITH MYSQL


![1](https://user-images.githubusercontent.com/114196715/195963990-982d2e4c-78d4-48ac-ace3-3d6ba6043924.png)

- Client-server model is a distributed application structure that partitions tasks or workloads between the providers of a resource or service, called servers, and service requesters, called clients.
- A server host runs one or more server programs, which share their resources with clients. A client usually does not share any of its resources, but it requests content or service from a server. 
- Clients, therefore, initiate communication sessions with servers, which await incoming requests.

![2](https://user-images.githubusercontent.com/114196715/195964012-d93e115e-94de-4846-aed2-d60a87fa3fd3.png)

- If we add a database to our server, we get the architecture below;

![3](https://user-images.githubusercontent.com/114196715/195964046-ef420882-a889-4c1e-8174-92f26255c63e.png)

* To demonstrate server-client communication, we shall run a curl command on a popular lamp stack website, [Propitixhomes](www.propitixhomes.com.) 

` curl -Iv www.propitixhomes.com `

- Here, our terminal is the client wherefrom requests originated to the server, www.propitixhomes.com.

## Create and configure two Linux-based virtual servers (EC2 instances in AWS)
- Server A name - 'mysql server'
- Server B name - 'mysql client'

![4](https://user-images.githubusercontent.com/114196715/195964109-8e159ce8-2586-49b9-9a17-6e06957022a3.png)

* To start with, I ran 'my_script', a simple shell script that I wrote to update and upgrade the package index of my server. It contains both ` sudo apt update ` and ` sudo apt upgrade `.

## On the MySQL server instance, install MySQL server package.

` sudo apt install mysql-server `

* Ensure the myql server is running using systemctl:

` sudo systemctl start mysql.service `

- The above commands will install and start mysql server but will not prompt you to set password or make other confuguration settings. 

- For added security in line with best practices, open up mysql console, run the ALTER USER command to change the root user’s authentication method to one that uses a password. This changes the authentication method to mysql_native_password.

` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; `

- Following this, you can decide to run the security script, ` sudo mysql_secure_installation `, validate password plugin if you want, and then complete the prompts. Thus, your myql server is secured.

- Create user, database and grant all permissions to the user:

```
CREATE USER 'Mukhtar'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

CREATE DATABASE Muku_DB;

GRANT ALL PRIVILEGES ON Muku_DB.* TO 'Mukhtar'@'%' WITH GRANT OPTION;

```
![5](https://user-images.githubusercontent.com/114196715/195964165-0b267d0c-fc87-4d0b-9b97-8675199802a9.png)

## Install MySQL Client software on the mysql client Linux Server

- MySQL is a popular open-source relational database management system.

- The program that interfaces with the server is known as a MySQL client; A shell that allows you to acess and manage your database remotely.

- The most basic client that one can use is the command line tool, most commonly known as MySQL client.

* After updating and upgrading your ubuntu, to install the client, run: ` sudo apt-get install mysql-client `

* Confirm if it was successfully installed. ` mysql -V `

* Now, you can connect to your remote database using ` mysql -u USERNAME -p PASSWORD -h HOST-OR-SERVER-IP `

* Note that mysql server should allow remote access to the server in order for the MySQL client to connect from a remote location. This, we shall do in the next step.*


## Configure MySQL server to allow connections from remote hosts:

- The default behavior of the Ubuntu MySQL Server blocks all remote connections which prevent us from accessing the database server from the outside.

- To allow mysql remote connections, we need to edit the MySQL main configuration file.

- For MySQL Database server, the configuration file is: "/etc/mysql/mysql.conf.d/mysqld.cnf".

- Open the '/etc/mysql/mysql.conf.d/mysqld.cnf' file

` sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf `

- Under the [mysqld] section, locate the line:

` bind-address            = 127.0.0.1 ` and change it to ` bind-address            = 0.0.0.0 `

- Save the configuration file, and restart the MySQL server:

` sudo systemctl restart mysql `

- To ensure connection, we shall edit the inbound rule of our mysql server security group to allow access from the IP of the mysql client (For security best practice)..

- We shall open up TCP traffic on port 3306 in the security group of our MySQL server.

![6](https://user-images.githubusercontent.com/114196715/195964262-3b7c62f4-5629-4568-8fe3-c7e5a7ba3a53.png)

## From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH.

* Connect to your mysql server by running the command below in terminal:

` mysql -u Mukhtar -h 172.31.85.208 -p `


## Check that you have successfully connected to a remote MySQL server and can perform SQL queries:

` Show databases; `

![7](https://user-images.githubusercontent.com/114196715/195964296-be188b33-0114-41bf-a9cf-28fe82136885.png)
