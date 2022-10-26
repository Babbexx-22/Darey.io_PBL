## PROJECT 6: WEB SOLUTION WITH WORDPRESS
- WordPress is a free and open-source content management system (CMS) written in hypertext preprocessor language[PHP] and paired with a MySQL or MariaDB database with supported HTTPS. 
- WordPress is a factory that makes webpages: it stores content and enables a user to create and publish webpages, requiring nothing beyond a domain and a hosting service.
- Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture; a client-server software architecture pattern that comprise of 3 separate layers:
1.Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
2.Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
3.Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

![1](https://user-images.githubusercontent.com/114196715/197939346-85951895-32e9-4842-a1ca-df5f2f1ae136.png)

In this project, we shall:

* Configure storage subsystem for Web and Database servers based on Linux OS. This part focuses on practical experience of working with disks, partitions and volumes in Linux.

* Install WordPress and connect it to a remote MySQL database server. This part will soldify our skills of deploying Web and DB tiers of Web solution.

## Step 1 — Prepare a Web Server

NOTE: I spinned up an ec2 instance with red hat linux OS AMI.

* Launch an EC2 instance that will serve as "Web Server". Create 3 EBS volumes in the same AZ as your Web Server EC2, each of 10 GiB size.
* Attach each of the EBS volumes to your web server instance.
* Open up the Linux terminal to begin configuration, ssh into your server.
* Use `lsblk` command to inspect what block devices are attached to the server. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there; their names will likely be xvdf, xvdh, xvdg.

![3](https://user-images.githubusercontent.com/114196715/197939696-8f491f7f-f760-4d9b-8ec0-9862e11482c9.png)

* Use `df -h` command to see all mounts and free space on the server.
* Use gdisk utility to create a single partition (10GB) on each of the 3 disks:

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh

```
* Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![2](https://user-images.githubusercontent.com/114196715/197939484-d962e433-285d-4ca5-a4c6-1c98d4eabca2.png)

* Install lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.
* Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

```
* Verify that the physical volumes were created successfully with ` sudp pvs `.

![5](https://user-images.githubusercontent.com/114196715/197939921-7d4d96c7-16a7-4feb-9a5f-df9a9a989a1c.png)

* Using the `vgcreate` utility, create a volume group named 'webdata-vg' and add all the three PVS to the volume group.

` sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 `

* Verify that the volume group was created successfully using ` sudo vgs `

![VGS](https://user-images.githubusercontent.com/114196715/197940011-8cd356a0-b4bd-4f61-bf1d-c508bea9907f.png)

* Using the `lvcreate ` utility, create two logical volumes(LVs); 'apps-lv' and 'logs-lv'. Assign each LVs about half of the volume group size which is about 30GB.

- I assigned both 14GB size; apps-lv will be used to store data for the Website while logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

``` 

* Verify that the LVs were created successfully using ` sudo lvs `.

![LVS](https://user-images.githubusercontent.com/114196715/197940107-a7187027-d5b9-4611-8e89-8fe167a6ed3f.png)

* Verify the entire setup:

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk

```

![LSBLK](https://user-images.githubusercontent.com/114196715/197940250-afdcb4f1-6190-4324-80ff-f9599085f4b9.png)

* Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

```

* Create /var/www/html directory to store website files:

` sudo mkdir -p /var/www/html `

* Create /home/recovery/logs to store backup of log data:

` sudo mkdir -p /home/recovery/logs `

* Mount /var/www/html on apps-lv logical volume:

` sudo mount /dev/webdata-vg/apps-lv /var/www/html/ `

* We shall mount 'logs-lv' logical volume on /var/log directory . Since this mount point isn't an empty directory, there is need to backup its content before mounting as mounting would cover up this content and will render it unavailable until one unmounts..

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs :

` sudo rsync -av /var/log/. /home/recovery/logs/ `

- Proceed to mount logs-lv logical volume on /var/log. (Note that all the existing data on /var/log will be covered up and rendered unavailable to whatever process uses them)

` sudo mount /dev/webdata-vg/logs-lv /var/log `

- Then, restore the log files back into /var/log directory:

` sudo rsync -av /home/recovery/logs/. /var/log `

TIP:
- Rsync is a free software utility for Unix- and Linux-like systems that copies files and directories from one host to another. 
- It is considered to be a lightweight application because file transfers are incremental. 
- After the initial full transfer, only bits in files that have been changed are transferred.

* To make persistent the mount configuration even after a system reboot, update the /etc/fstab file.

- The UUID of the device will be used to update the /etc/fstab file;

```
 sudo blkid
 sudo vi /etc/fstab

``` 

![fstab](https://user-images.githubusercontent.com/114196715/197940468-755296f3-3d20-4252-9572-66ee53a1051f.png)

* Test the configuration and reload the daemon. The ` mount -a ` command checks to ensure that no error was made while updating the fstab file.

```
 sudo mount -a
 sudo systemctl daemon-reload

```

* Verify the setup by running ` df -h `

![df](https://user-images.githubusercontent.com/114196715/197940667-ba29f24f-e559-4102-bed6-738624db4476.png)

## Step 2 — Prepare the Database Server

*  A second RedHat EC2 instance was launched and the above steps were repeated. instead of 'apps-lv' logical volume, 'db-lv' was created and mounted to '/db' directory instead of /var/www/html directory.


## Step 3 — Install WordPress on the Web Server EC2

* After updating the server ` sudo yum update -y `, install wget, Apache and its dependencies

` sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json `

* Start Apache

```
 sudo systemctl enable httpd
 sudo systemctl start httpd

```

* To install PHP and it’s dependencies using REMI repository

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1

```

* Restart Apache

` sudo systemctl restart httpd `

* Download wordpress and copy wordpress to var/www/html

```
  mkdir wordpress
  cd  wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/

```

* Configure SELinux Policies

```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1

```


## Step 4 — Install MySQL on the DB Server EC2

* Update and install mysql server:

``` 
 sudo yum update
 sudo yum install mysql-server

```

* Verify that the service is up and running by using `sudo systemctl status mysqld `.


## Step 5 — Configure DB to work with WordPress.

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER 'myuser'@'172.31.84.166' IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.84.166';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```

## Step 6 — Configure WordPress to connect to remote database.

- Since our web server housing wordpress will be required to connect to our database server, ensure to configure the database server to allow remote access by editing/adding the bind address (set to 0.0.0.0) in the mysql configuration directory; '/etc/my.cnf'.

` sudo vi /etc/my.cnf `

- Add : 	[mysqld]
		bind-address=0.0.0.0

- We should also open mysql port 3306 on the database server to allow traffic from the private IP of our web server.

* Install mysql client on the web server and test that you can connect to the database server.

` sudo yum install mysql `
` sudo mysql -u admin -p -h <DB-Server-Private-IP-address> `

* Verify that you can execute the 'SHOW DATABASES;' to access the list of database therein.

![database](https://user-images.githubusercontent.com/114196715/197940836-89ac01a1-54ec-4eb7-9758-5991e64902ce.png)

* Change permissions and configuration so Apache could use WordPress

` sudo chown -R apache:apache /var/www/html/wordpress `

* From your browser, try to access the link to your WordPress ` http://<Web-Server-Public-IP-Address>/wordpress/ `

![web](https://user-images.githubusercontent.com/114196715/197941580-b9257ccd-f51e-49ff-b1ad-fa0e9aee9fa4.png)

NB: 
- WordPress stores our database information in the wp-config.php file, a configuration file that is part of all self-hosted WordPress sites.
- Unlike other files, wp-config.php file does not come built-in with WordPress rather it’s generated specifically for your site during the installation process.
- Without this information your WordPress website will not work, and you will get the ‘error establishing database connection‘ response.
- The wp-config.php file should be opened and the required database information should be updated.

![Final](https://user-images.githubusercontent.com/114196715/197941720-2b14aa16-98a2-4cdc-88d3-483c2370ba30.png)

