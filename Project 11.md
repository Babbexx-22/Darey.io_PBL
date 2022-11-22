## TEST AUTOMATION WITH ANSIBLE: AUTOMATE PROJECT 7 TO 10

Ansible is an open source, command-line IT automation software application written in Python. It can configure systems, deploy software, and orchestrate advanced workflows to support application deployment, system updates, and more.

Ansible’s main strengths are simplicity and ease of use. It also has a strong focus on security and reliability, featuring minimal moving parts. It uses OpenSSH for transport (with other transports and pull modes as alternatives), and uses a human-readable language (YAML) that is designed for getting started quickly without a lot of training.

The architecture of our setup is as shown below:

![archy](https://user-images.githubusercontent.com/114196715/203436509-9ed4d85e-d7c4-4f6e-9c6e-dfd4c6f2351e.png)

ANSIBLE CLIENT AS A JUMP SERVER (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. In this project, we shall:
- set up ssh-agent to store our key pairs and render them for use when needed.
- Use our Ansible server as a bastion host through we shall be able to ssh into our webservers (ideally) locked away in the private subnets using their individual private IPs.
- Create a simple Ansible playbook to automate servers configuration; Exhibit test automation to run configurations on our diffrent servers without leaving our ansible server.
- Configure Jenkins build job to save our repository content every time changes are made.
- explore git version control actions; clone repository, create branches to simulate production environment, run git status, add changes to the staging index, run git commits, push our changes to github, create pull requests and ultimately merge changes.



## STEP 1: INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

2. In your GitHub account create a new repository and name it Ansible-config

3. Install Ansible

``` 
sudo apt update 
sudo apt install ansible

```
Check your Ansible version by running ` ansible --version `

![ansible version](https://user-images.githubusercontent.com/114196715/203436616-87d0d8aa-f0a8-4006-aa29-ef4c34b318e9.png)

4. Configure Jenkins build job to save your repository content every time you change it.

- Create a new Freestyle project 'ansible' in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![scm git](https://user-images.githubusercontent.com/114196715/203436695-34e92f93-a41b-4bce-8d71-fabfa287002c.png)

- Configure Webhook in GitHub and set webhook to trigger ansible build.

![webhook](https://user-images.githubusercontent.com/114196715/203436777-0b131ef8-b215-48a8-8868-68bc0efc150e.png)

- Configure a Post-build job to save all (**) files.

![post build actions](https://user-images.githubusercontent.com/114196715/203436846-ba5433da-f6af-49ac-ac80-e2629ebee917.png)

- Test your setup by making some changes in README.MD file in master branch and make sure that builds starts automatically. Jenkins saves the files (build artifacts) in the following folder ` ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ `


## STEP 2 – PREPARE YOUR DEVELOPMENT ENVIRONMENT USING VISUAL STUDIO CODE.

* configure VS Code to connect to your newly created GitHub repository.

* Clone your Ansible repository so you can begin work on it.

## STEP 3: BEGIN ANSIBLE DEVELOPMENT

1. In your ansible-config GitHub repository, create a new branch that will be used for development of a new feature.

I did this on the vs code terminal using ` git checkout -b <name of branch> `. This command will create the specified branch and as well checkout to the branch. To confirm this, run ` git branch `. The checkout branch bears an asterik just before it.

![branch](https://user-images.githubusercontent.com/114196715/203436943-174d1453-2bf9-4a53-bdaa-9332b2b658dd.png)

2. Create a directory and name it 'playbooks' – it will be used to store all your playbook files.

3. Create a directory and name it inventory – it will be used to keep your hosts organised.

4. Within the playbooks folder, create your first playbook, and name it common.yml

5. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

![vs code ansible](https://user-images.githubusercontent.com/114196715/203437080-f1b8a9fa-2a60-4dfb-a997-bd89c70ae626.png)

* Set up ssh agent, add key pair and ssh into your jenkins-ansible server using ssh agent. 

On my local system, I installed open-ssh server and client as well as ssh agent. This was followed by adding my aws key pair to ssh agent using `ssh-add <key pair>` and confirmed with `ssh-add -l`. Since this is the key pair used in creating all my servers, SSH into these servers is made easy by simply using `ssh -A ubuntu@public IP 0r ssh -A ec2-user@public IP ` depending on the linus AMI used. Thus, ssh agent renders the key pair for use when needed.

- Now, ssh into your Jenkins-Ansible server using ssh-agent

![ssh with ssh agent](https://user-images.githubusercontent.com/114196715/203437195-4c6e29c8-4eef-4416-9019-f2103177a5b3.png)

- Therefrom exhibit bastion host to ssh into other servers using their private IPs.

![bastion db](https://user-images.githubusercontent.com/114196715/203437326-43fa0729-756c-45d4-a319-f212f26bffc7.png)

![bastion nfs](https://user-images.githubusercontent.com/114196715/203437503-982b8ae3-e171-4c26-a03c-e3f7e8c52835.png)

## Step 4 – SET UP AN ANSIBLE INVENTORY

1. Update your inventory/dev.yml file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```

![dev yml](https://user-images.githubusercontent.com/114196715/203437595-d0a8b982-0acf-4ce7-96ec-8afb9803a95c.png)

## Step 5 – CREATE A COMMON PLAYBOOK

In common.yml playbook, we will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

* Update your playbooks/common.yml file with following code:

```
---
- name: update web and nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB and DB server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```

![inventory playbooks](https://user-images.githubusercontent.com/114196715/203437672-a1e296e1-a5c7-436b-bce6-5a3688a62196.png)

## Step 6 – UPDATE GIT WITH THE LATEST CODE

* Use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"

git push origin Project11

```

* Create a Pull request (PR) and merge the code to the master branch.

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to ` /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ ` directory on Jenkins-Ansible server.

![build in jenkins]https://user-images.githubusercontent.com/114196715/203437813-975f479b-4df1-4030-a80c-a5d5b5bb0a95.png)

## Step 7 – RUN FIRST ANSIBLE TEST

Now that the repository is well updated with the required folders and files, ssh into your jenkins-ansible server and clone the repository.

* Change directory to the cloned folder (` cd Ansible-config`) and run the playbook command ` ansible-playbook -i inventory/dev.yml playbooks/common.yml `

![nfs,web playbook](https://user-images.githubusercontent.com/114196715/203437910-e0fb0642-5982-4884-b8fe-1ee33403993f.png)

CHALLENGE:

From the image above, it is seen that only the nfs server and web servers had wireshark installed successfully. There was an unending request while trying to intall for both lb and db servers. In an attempt to solve this, I used this command ` ansible-playbook -i inventory/dev.yml playbooks/common.yml --limit "lb' ` to selectively run the playbook on my lb server.

![lb playbook](https://user-images.githubusercontent.com/114196715/203438039-2b2fa519-529b-4de0-9ff6-b8bc52947db9.png)

Running the playbook ` ansible-playbook -i inventory/dev.yml playbooks/common.yml ` a second time however installed wireshark on my db server. Upon subsequent unending running of the playbook, simply alting the process with "control c" and running the playbook solved the encounter.

![nfs,web,lb,db](https://user-images.githubusercontent.com/114196715/203438131-221369da-8fd7-4983-9a73-be459bdb75ae.png)

## Optional steP

- I updated my playbook with a command to install httpd on my servers.

- I used ansible.builtin.package module this time around to boycott having to specify yum or apt in our playbook.

- This module manages packages on a target without specifying a package manager module (like ansible.builtin.yum, ansible.builtin.apt, …). It is convenient to use in an heterogeneous environment of machines without having to create a specific task for each package manager. This makes work easier especially when one is working with hundreds of servers of which you do not know the specific linux distro used in spinning them.

Here's the snippet of code I updated my playbook with:

```
FOR NFS and WEB SERVERS

- name: install httpd
  ansible.builtin.package:
    name: httpd
    state: latest

FOR LB AND DB SERVER

- name: install apache2
  ansible.builtin.package:
    name: apache2
    state: latest

```
![package httpd 1](https://user-images.githubusercontent.com/114196715/203438233-495867dc-c873-4715-8582-9a51d140f76f.png)

![package httpd 2](https://user-images.githubusercontent.com/114196715/203438295-268ac1b4-744b-4147-bdff-433041798079.png)

N.B: As more and more changes are made in our repository and pushed to github, Jenkins automatically log the changes in `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory with the build number changing with subsequent push actions. Changing directory to the latest build number folder before running our playbook is important as that is where our updated changes are logged.

* The playbok was run in the appropriate jenkins build log folder and apache was successfully installed on all five instances.

![httpd overall](https://user-images.githubusercontent.com/114196715/203438407-0868a273-5986-41e9-a1c4-da347bae5c76.png)

----------------------
THE END
