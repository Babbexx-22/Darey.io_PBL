## TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION; INTRODUCTION TO JENKINS

DevOps (a portmanteau of “development” and “operations”) is the combination of practices and tools designed to increase an organization’s ability to deliver applications and services faster than traditional software development processes. This speed enables organizations to better serve their customers and compete more effectively in the market.

Continuous integration is a software development method where members of the team can integrate their work at least once a day. In this method, every integration is checked by an automated build to search the error. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

Jenkins is an open-source Continuous Integration server written in Java for orchestrating a chain of actions to achieve the Continuous Integration process in an automated fashion. 

In this project, we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub; "https://github.com/Babbexx-22/tooling" will be automatically updated to the Tooling Website.


*Task: Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

### INSTALL AND CONFIGURE JENKINS SERVER

STEP 1: Install Jenkins server

* Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
* Install JDK as Jenkins is a Java-based application:

```
sudo apt update
sudo apt install default-jdk-headless

```

* Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins

```

* Ensure Jenkins is up and running: ` sudo systemctl status jenkins `

* Jenkins uses TCP Port 8080, edit the security group of the ec2 instance and allow traffic form the said port.

![Security_Group](https://user-images.githubusercontent.com/114196715/202042551-6140f684-660d-483c-b7cd-cf6674e99d16.png)

* Perform initial Jenkins set-up: From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080, you will be prompted to provide a default admin password. Retrieve it from your server: ` sudo cat /var/lib/jenkins/secrets/initialAdminPassword `.

![password](https://user-images.githubusercontent.com/114196715/202042884-788c18f3-86a2-4ec3-981e-2300b67faaf5.png)
  
![jenkins  web](https://user-images.githubusercontent.com/114196715/202042706-dd933e30-8690-47bc-951d-73a2244d75d3.png)
  
* When asked which plugins to install, choose suggested plugins.

![jenkis homepage](https://user-images.githubusercontent.com/114196715/202043047-f8269fb8-e12c-4692-b6a7-12379f97d104.png)
  
* Once plugins installation is done – create an admin user and you will get your Jenkins server address.The installation is completed!

![welcome page](https://user-images.githubusercontent.com/114196715/202044304-40c3d93c-d9cb-4c59-8148-10ad7775132e.png)
  
STEP 2 : Configure Jenkins to retrieve source codes from GitHub using Webhooks.

In this part, we wil configure a simple Jenkins job. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings.

![add webhook](https://user-images.githubusercontent.com/114196715/202043361-cba5533f-9607-432a-b199-a8a1d51d3d60.png)
  
2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![new build](https://user-images.githubusercontent.com/114196715/202043525-11b52fef-c341-41fc-8bcf-323cc17593d3.png)
  
To connect to our github repository, we need to provide its URL which can be obtained from the repository.

![repo url](https://user-images.githubusercontent.com/114196715/202043995-fcdd447b-15bb-4c8f-a139-bdcfe3db1f94.png)

* While configuring the Jenkins freestyle job, choose git in the source code management page, provide the link to your tooling github repository alongside your credentials (username and password) so that Jenkins can access files in the repository.

* Save the configuration file and try to run the build. We shall do this manually; click on 'build now' option. Open the build and check in "Console Output" if it has run successfully.

![build now](https://user-images.githubusercontent.com/114196715/202044528-84e399cb-da59-4cb1-a177-06231bab1bb4.png)
  
3. Click "Configure" your job/project and add these two configurations.
	- Configure triggering the job from GitHub webhook
	- Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

To test this configuration, a change is made in any file in the github repository and changes pushed to the master branch. it is observed that a new build has been launched automatically (by webhook) and one can see its results – artifacts, saved on Jenkins server.

![build 2](https://user-images.githubusercontent.com/114196715/202045576-f493c6df-18ed-417f-8a52-89fc2bce1680.png)
  
By default, the artifacts are stored on Jenkins server locally ` ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/ `

![artifacts stored in jenkins](https://user-images.githubusercontent.com/114196715/202044710-17176f02-5cb4-4c45-bb2a-00d1c5dad05d.png)
  
## STEP 3 – Configure Jenkins to copy files to NFS server via SSH

Now that we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.

* On the main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item;
* On "Available" tab search for "Publish Over SSH" plugin and install it
  
![pub over ssh plugin](https://user-images.githubusercontent.com/114196715/202044924-e2c35578-51d6-47eb-804d-90e0d6a9a0cb.png)
  
2. Configure the job/project to copy artifacts over to NFS server.

* On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

* Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to our NFS server:

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since our NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server.

 ![nfs ssh configuration](https://user-images.githubusercontent.com/114196715/202045117-ac93b80a-f737-4088-9313-b8f00c24cdb9.png)
  
* Test the configuration and make sure the connection returns Success. TCP port 22 on NFS server must be open to receive SSH connections.

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action" (Send build artifacts over SSH)

* Configure it to send all files produced by the build into our previously defined remote directory. In our case we want to copy all files and directories – so we use **.

![post build actions](https://user-images.githubusercontent.com/114196715/202045425-9dcd4122-e409-4ade-8d3e-53b43f3d7359.png)
  
* Save this configuration and go ahead to change something in README.MD file in your GitHub Tooling repository. Webhook will trigger a new job and a report of the job is found in the Console Output.

![nfs build successful](https://user-images.githubusercontent.com/114196715/202045834-8bb1b187-c49c-4002-93e9-ce5c079491ea.png)
  
* To confirm that the changes made to the readme file has been updated, SSH into the nfs server and check the content of the readme file: ` sudo nano /mnt/apps/README.md `.

![NFS BUILD EFFECTED](https://user-images.githubusercontent.com/114196715/202045939-b4729f4c-0d95-4022-8d16-197423da9863.png)
