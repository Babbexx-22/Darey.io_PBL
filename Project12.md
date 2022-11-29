## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

Refactoring is the process of restructuring code, while not changing its original functionality. The goal of refactoring is to improve internal code by making many small changes without altering the code's external behavior. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

There are two ways by which Ansible is able to reuse our configuration management tasks within playbooks; "import and include" and roles. Both work in a similar way, but roles allow you a lot more flexibility. Instead of simply packaging tasks together, roles allow you to set up a specific structure to include variables, handlers, modules, and other plugins, all of which can be uploaded to Ansible Galaxy which is a central repository or hub for sharing, finding, and reusing your Ansible code. 

Both include and import do similar things, but import statements are preprocessed (static) at the time the playbook is parsed by Ansible , whereas the included statements are processed as they are encountered during the execution of the playbook.

The architecture of the project is as shown below;

![architecture](https://user-images.githubusercontent.com/114196715/204665517-0125d75e-6839-46d3-9e18-c9fd9374d473.png)

## Step 1 – Jenkins job enhancement

Using the copy artifact plugin, we shall create a new directory wherein our jenkin jobs will be saved. This will enhance convenience while allowing us to run our playbook in one place as opposed to the differing log points used earlier secondary to change in build numbers in /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ ` directory.


1. Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store therein all artifacts after each build.

` sudo mkdir /home/ubuntu/ansible-config-artifact `

2. Change permissions to this directory, so Jenkins could save files there – ` chmod -R 0777 /home/ubuntu/ansible-config-artifact `

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![copy artifact](https://user-images.githubusercontent.com/114196715/204665853-3fb9015b-cbe4-480f-9289-7be846bc7309.png)

4. Create a new Freestyle project and name it "save_artifacts".

5. This project will be triggered by the completion of our existing ansible project. It was configured as shown below:

![log rotation](https://user-images.githubusercontent.com/114196715/204667114-482f8d29-bbfa-42ef-b297-ebe29e16efce.png)

![build trigger](https://user-images.githubusercontent.com/114196715/204666377-a1ba1d5e-8c8a-4c59-ad0b-dc0b5486bd2e.png)


6. To save our artifacts into "/home/ubuntu/ansible-config-artifact" directory, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![build steps](https://user-images.githubusercontent.com/114196715/204666479-eafa1961-2824-4fef-86a1-ca837bf9a019.png)

7. Test the set up by making some changes in README.MD file inside your ansible-config repository (right inside master branch). If both Jenkins jobs have completed one after another – we shall see our files inside "/home/ubuntu/ansible-config-artifact" directory and it will be updated with every commit to our master branch.

![artifact saved in instance](https://user-images.githubusercontent.com/114196715/204666701-2035f86f-7979-488f-8f1c-cd9b0914d97e.png)

## STEP 2 – REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOK INTO site.yml

Ensure that you have pulled down the latest code from master (main) branch, and create a new branch, name it "refactor".

1. Within playbooks folder, create a new file and name it "site.yml" – This file will now be considered as an entry point into the entire infrastructure configuration. 

2. Create a new folder in the root of the repository and name it "static-assignments". The "static-assignments" folder is where all other children playbooks will be stored.

3. Move common.yml file into the newly created "static-assignments" folder.

4. Inside site.yml file, import common.yml playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

```

The folder structure looks like below:

![folder structure](https://user-images.githubusercontent.com/114196715/204666900-75359366-a053-471a-92a0-d51d99fdcf65.png)

5. Run ansible-playbook command against the dev environment. 

To effect this, we shall write another playbook to uninstall wireshark first after which we shall run the earlier created playbook; "site.yml".
"
- create another playbook under "static-assignments" and name it "common-del.yml". In this playbook, configure deletion of wireshark utility:

```
---
- name: update web and nfs
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB and db servers
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes

```

- update site.yml with;

` - import_playbook: ../static-assignments/common-del.yml ` instead of common.yml 

- Run the site.yml playbook against the dev servers:

```
cd /home/ubuntu/ansible-config-artifact/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml

```

![wireshark uninstalled](https://user-images.githubusercontent.com/114196715/204667332-c3602d93-cc46-4b05-a756-5751b430cb3e.png)

Confirm that wireshark is deleted on all the servers by running `wireshark --version`.

![wireshark uninstalled check](https://user-images.githubusercontent.com/114196715/204667437-7500a940-d31a-4731-803a-400043476401.png)

## STEP 3 – CONFIGURE UAT WEB SERVERS

We shall configure 2 new Web Servers to serve as our testing environment (UAT).

1. Launch 2 new EC2 instances using RHEL 9 image, we will use them as our uat servers; Web1-UAT and Web2-UAT

2. To create a role, we shall create a directory called "roles/" inside our Ansible-config folder, relative to the playbook file or in /etc/ansible/ directory.

As above, this can be done in two ways;

- Use an Ansible utility called ansible-galaxy inside Ansible-config/roles directory (you need to create roles directory upfront)

```
mkdir roles
cd roles
ansible-galaxy init webserver

```
	OR

- Create the directory/files structure manually.

![tree](https://user-images.githubusercontent.com/114196715/204667689-36f6b287-b145-43eb-88dd-8eacd7eb4b26.png)

3. Update the inventory (ansible-config-mgt/inventory/uat.yml) file with the private IP addresses of our 2 UAT Web servers.

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

```

![uat](https://user-images.githubusercontent.com/114196715/204667908-9873ec29-0a00-4ee2-907b-79bdaf25b37f.png)

4. In "/etc/ansible/ansible.cfg" file uncomment roles_path string and provide a full path to your roles directory; roles_path = /home/ubuntu/ansible-config-artifact/roles, so Ansible could know where to find configured roles.

NOTE: Due to an issue I encountered with my copy artifact plugin(wherein subsequent builds wasn't copied successfully), I had to change my role path to /var/lib/jenkins/jobs/Ansible/builds/20/archive/roles with the number "21" representing the latest jenkins build number secondary to my github push. I resorted to this step as my playbook threw an error indicating that it couldnt find my role "webserver" in the former declared location "/home/ubuntu/ansible-config-artifact/roles".

![my role path](https://user-images.githubusercontent.com/114196715/204668034-604452a1-b924-4597-9811-8e1d4b54bd14.png]

5. within the main.yml file in the tasks directory, write configuration tasks to do the following:

- Install and configure Apache (httpd service)
- Clone Tooling website from GitHub https://github.com/Babbexx-22/tooling_.git.
- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
- Make sure httpd service is started.

The main.yml file will thus have the following content;

```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: git
    state: present

- name: clone tooling repository
  become: true
  ansible.builtin.git:
    repo: https://github.com/Babbexx-22/tooling.git.
    dest: /var/www/tooling
    force: yes

- name: copy html content from /var/www/tooling/html/ to /var/www
  become: true
  command: cp -r /var/www/tooling/html/ /var/www/

- name: start httpd if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent

```
The provided code (meant to be used above) was found to be glitchy and was thus modified. Also, I tried to use the Ansible.builtin.copy module to perform the copy task above;

```
- name: copy html content from /var/www/tooling/html/ to /var/www
  become: true
  ansible.builtin.copy:
    src: /var/www/tooling/html
    dest: /var/www

```
I however was met with the error below:

![error](https://user-images.githubusercontent.com/114196715/204668264-bd9be978-bf3d-4848-b913-6e73e141fc94.png)

I shall go back to practice ansible the more and perform further troubleshooting.

The playbook tasks were sucessfully executed

![all installed](https://user-images.githubusercontent.com/114196715/204668526-db657957-7dd4-4827-b231-b164ac4c7ee4.png)

## STEP 4 – REFERENCE ‘Webserver’ ROLE

* Within the static-assignments folder, create a new assignment called "uat-webservers.yml" for our uat-webservers . This is where we will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver

```

Since the entry point to our ansible configuration is the site.yml file. Therefore, we need to refer our uat-webservers.yml role inside site.yml. Hence, the site.yml file will contain the following:

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

```

## STEP 5 - COMMIT AND TEST

- Commit your changes, create a Pull Request and merge them to master branch.
- Make sure webhook triggered two consequent Jenkins jobs; ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-artifact/ directory.

NOTE: The above was not successful. The copy artifact plugin worked for a while and then stopped copying the updated build at around build #9. I however simulated the intended refactoring in step 2 while the plugin was still intact and do attest to the ease that comes with using the copy artifact plugin.

* Run the playbook against your uat inventory 

` sudo ansible-playbook -i /var/lib/jenkins/jobs/Ansible/builds/20/archive/inventory/uat.yml /var/lib/jenkins/jobs/Ansible/builds/20/archive/playbooks/site.yaml `

Our UAT Web servers was configured and we can try to reach them from our browser:

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

or

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php


![website successful](https://user-images.githubusercontent.com/114196715/204668703-1923af99-97b0-4f0a-9ed6-1af2ef89760a.png)
  
---------------------------------------------------------------
*THE END*  
