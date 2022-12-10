## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

When the import module is used, all statements are pre-processed at the time playbooks are parsed. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

Including roles, tasks, or variables adds them to a playbook dynamically. Ansible processes included files and roles as they come up in a playbook, so included tasks can be affected by the results of earlier tasks within the top-level playbook. Included roles and tasks are similar to handlers - they may or may not run, depending on the results of other tasks in the top-level playbook.

The primary advantage of using include_* statements is looping. When a loop is used with an include, the included tasks or role will be executed once for each item in the loop.

In most cases however, it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature.


## STEP 1: Introducing Dynamic Assignment Into Our structure

* In your https://github.com/<your-name>/ansible-config-mgt GitHub repository, start a new branch and call it "dynamic-assignments".

* Therein, create a new folder, name it "dynamic-assignments" and create a file in the new folder "env-vars.yml"

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. Thus, we shall create a folder to keep each environment’s variables file.

* create a new folder "env-vars", and for each environment (dev,stage,uat and prod), create new YAML files which we will use to set variables.

The tree structure thus looks like the below:

![tree](https://user-images.githubusercontent.com/114196715/206763925-46e990b5-f29a-4b04-bf45-2e63932624ac.png)
  
* Update the env-vars.yml file (dynamic-assignments/env-vars.yml) with the code below:

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always

```

We made use of a special variables "{ playbook_dir }", this will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. "{ inventory_file }" on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the 'env-vars' folder.

We are including the variables using a loop 'with_first_found' which implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.


## STEP 2: UPDATE 'SITE.YML' WITH DYNAMIC ASSIGNMENTS

* Update site.yml file to make use of the dynamic assignment. Our site.yml file should now look like below:

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/uat-webservers.yml

```


In order to sync our github with the jenkins server directly, we shall configure our VS code to work directly with github;

* On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run

```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature

```

![git ](https://user-images.githubusercontent.com/114196715/206764071-97e93c0e-98cb-4b80-9904-8e382305f277.png)
  
## STEP 3: Community Roles

Next is to create a role for MySQL database – it should install the MySQL package, create a database and configure users. Instead of re-inventing the wheel however, we shall use roles already developed by open source engineers. With Ansible Galaxy, we can simply download a ready to use ansible role, and keep going.

We will be using a MySQL role developed by 'geerlingguy'.

* Inside roles directory , create your new MySQL role with `ansible-galaxy install geerlingguy.mysql` and rename the folder to mysql;

` mv geerlingguy.mysql/ mysql `

![mysql role](https://user-images.githubusercontent.com/114196715/206764213-36830a99-cfb2-4a10-ad3c-52ec99d2bca4.png)
  
* Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

The point wherein changes were made in the mysql role is shown below:

* The main.yml file in the defaults folder: the database name was modified to "tooling" and the user to "webaccess". The configuration was set such that the user has access to the database , a privilege set with GRANT option. 

![mysql db and user](https://user-images.githubusercontent.com/114196715/206764441-fbb37d18-9a48-4105-9116-dd816f32db91.png)
  
* upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature 

```

* Create a Pull Request and merge it to main branch on GitHub.


## STEP 4: LOAD BALANCER ROLES.

In order to be able to chose which loadbalancer to use (Apache or nginx), we need to have two roles respectively; Apache and Nginx.

* Using the earlier introduced community role; geerlingguy, cd to the role directory, run `ansible-galaxy install geerlingguy.apache` and `ansible-galaxy install geerlingguy.nginx`.

* rename the role to apache and nginx respectively; 
```
mv geerlingguy.apache/ apache
mv geerlingguy.nginx/ nginx

```
![apache nginx role](https://user-images.githubusercontent.com/114196715/206774358-9f0db3a0-e68e-4125-9d2a-e389dbbc4ccb.png)
  
* Update both static-assignment and site.yml files to refer the roles.

- The static assignment folder was updated to contain db.yml file and lb.yml file.

The db.yml file is as below;
![db yml](https://user-images.githubusercontent.com/114196715/206764587-acfe2cba-2cc8-4934-b3fb-ceb3bfd791f2.png)
 
The lb.yml file is as shown below;
![lb yml](https://user-images.githubusercontent.com/114196715/206764806-7b0eb900-f0c1-4ef6-aacb-e723b869eb2c.png)
  
- The site.yml file was then updated to refer the above two playbooks (db.yml and lb.yml) as well as the env-vars.yml file.  A copy of the file is attached below;

![site yml](https://user-images.githubusercontent.com/114196715/206764925-db2f8cf3-4bc3-4d08-b2d8-ad56590ee4e7.png)
  
* Since we cannot use both Nginx and Apache load balancer, we shall add a condition to enable either one – this will be done by making use of variables.

- Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables 'enable_nginx_lb' and 'enable_apache_lb' respectively.

- Set both values to false

- Declare another variable in both roles called 'load_balancer_is_required' and set its value to false as well.

The snippet below is placed in the nginx roles' defaults/main.yml file.

```
enable_nginx_lb: false
load_balancer_is_required: false

```

while the below is placed in Apache's defaults/main.yml file.

```
enable_apache_lb: false
load_balancer_is_required: false

```

- We will then use 'env-vars\uat.yml' file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

* Activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

```
enable_nginx_lb: true
load_balancer_is_required: true

```
The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, you can update inventory for each environment and run Ansible against each environment.


## LB CONFIGURATION SETTINGS FOR APACHE ROLE.

From our knowledge of loadbalancing with apache in the previous project, the following points are crucial when configuring apache as a loadbalancer;
We shall highlight the points to guide us to where necessary modifications will be made in the downloaded roles. 

### INSTALLING APACHE AND STARTING THE SERVICE

This will be performed by default as part of the pre-configured settings in the apache role that was downloaded.


  
### ENABLING THE FOLLOWING MODULES

```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

```

  This will be included in the main.yml file present in the tasks directory of the role. The modules configurations were formatted in an ansible snippet format as shown below;  

```
- name: enable configurations
  become: true
  shell: a2enmod rewrite proxy proxy_balancer proxy_http headers lbmethod_bytraffic
  notify: restart apache

```
and carefully placed within the "#configure Apache" section present in the file:

![lb apache dependencies task](https://user-images.githubusercontent.com/114196715/206765193-a578b739-9f75-4951-a165-2d61a692f939.png)
  

### APACHE VIRTUAL HOST CONFIGURATION IN THE SITES-AVAILABLE DIRECTORY;

```
	  <Proxy "balancer://myapp1">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests

        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://myapp1/
        ProxyPassReverse / balancer://myapp1/


```

The apache virtual host configuration for loadbalancing shown above was placed in the "templates/vhosts.conf.j2" file in the section "{% if vhost.documentroot is defined %}". See below

![vhost conf j2](https://user-images.githubusercontent.com/114196715/206765398-b85af287-966e-48ca-9ca9-2f5443fe67b9.png)


Other modifications that will be done in the defaults/main.yml file of the apache role are noted below ;
 
- Define the loadbalancer name (server group) in the apache vhost section as shown below:

![lb name defined](https://user-images.githubusercontent.com/114196715/206765633-cae8f880-e958-487c-bf62-23846b25407e.png)
      
- Set the apache loadbalancer conditions to "true"

![lb condition and lb defined](https://user-images.githubusercontent.com/114196715/206765708-ebea4b80-727d-4b21-bbd6-57c9ed44f756.png)
      
- Remove the default apache configuration page by setting "apache_remove_default_vhost: true"

![remove default apache](https://user-images.githubusercontent.com/114196715/206765781-d49970f8-b5a7-47ff-bfb6-c3f037108074.png)

Finally, enable the apache loadbalacer conditions that was defined in env-vars/uat.yml file by setting the variable to true.

![env-vars](https://user-images.githubusercontent.com/114196715/206765916-c1f0ce7c-c22d-46bb-a051-28d7baba7275.png)
      
Above, the nginx condition was commented out. The converse will apply when we are set to use nginx as a loadbalancer.


* The playbook was run "ansible-playbook -i inventory/uat.yml playbooks/site.yml" in my "Ansible-congif" directory and the output is shown below:

![lb play 1](https://user-images.githubusercontent.com/114196715/206766108-4c8f61ae-bdda-46ec-9ed1-eddeff610d6d.png)

![lb play 2](https://user-images.githubusercontent.com/114196715/206766112-96dff072-34b2-4367-b0da-c7f768df7884.png)
      
![lb play 3](https://user-images.githubusercontent.com/114196715/206766102-5e5527f4-0305-43c2-9bb7-3d1860979adb.png)

* The public IP of the loadbalancer instance was rendered against our web browser

![nginx final lb site](https://user-images.githubusercontent.com/114196715/206767757-d231c182-6bc1-4f4a-9f16-6cf358079149.png)
      
* To ascertain the load balancing action of apache, the access_log file of the two web server instances were checked `sudo tail -f -n8 /var/log/httpd/access_log` to see how traffic requests were efficiently loadbalanced.

![web 1 log](https://user-images.githubusercontent.com/114196715/206767868-764bb5ad-0b1f-4db5-91d5-f1d4ee0ba9ab.png)
      
![web 2 log](https://user-images.githubusercontent.com/114196715/206768194-4c9c9267-63d7-483b-afe8-6bd562efefcb.png)
      
---------------------END OF APACHE LB------------------------------------


## LB CONFIGURATION SETTINGS FOR NGINX ROLE.

From our knowledge of loadbalancing with Nginx in the previous project, the following points are crucial when configuring nginx as a loadbalancer;

### Updating the '/etc/hosts' file to include the web servers’ names (e.g. Web1 and Web2) and their private IP addresses.

This was done using Ansible blockinfile module. The module is used to insert, update or remove a block of lines (multi-line text) from files on the remote nodes. These blocks are surrounded by the marker like begin and end which can be a custom marker. The module helps to work with the different types of files.

The code below was thus included in tasks/main.yml file within the vhost configuration section.

```
- name: set webservers host name in /etc/hosts
  become: yes
  blockinfile: 
    path: /etc/hosts
    block: |
      {{ item.ip }} {{ item.name }}
  loop:
    - { name: web1, ip: 172.31.83.5 }
    - { name: web2, ip: 172.31.86.8 }

``` 

      
### Editing nginx configuration file "/etc/nginx/nginx.conf" to include the upstream server and other http options.

The following was added in the defaults/main.yml file of our nginx role

```
upstream myapp1 {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name <Private IP of lb server>;
    location / {
      proxy_pass http://myapp1;
    }
  }

```

![nginx upstream](https://user-images.githubusercontent.com/114196715/206768435-bfb684b3-4986-430f-a32d-f932d13596bb.png)
      
      
### Enable loadbalancer using nginx by setting the variable to true in the same defaults/main.yml file

![enable nginx lb](https://user-images.githubusercontent.com/114196715/206768580-7023bc9b-b6be-4088-9cc5-40819b1ef2ca.png)
      
      
### In the same defaults/main.yml file, enable nginx extra http options by uncommenting the said section as shown below. Also, remove the default nginx page by setting the remove default page to true. 
 
![EXTRA HTTP OPTIONS](https://user-images.githubusercontent.com/114196715/206768665-5325301a-3a84-40e1-9de5-9653562c2767.png)
      

### In templates/nginx.conf.j2, include the server block configuration as shown below;

![server block nginx](https://user-images.githubusercontent.com/114196715/206769120-e5e5fc0a-1d70-42d0-92dd-a2cf91136eec.png)
      

### Finally, set the nginx loadbalancer variable to true in env-vars/uat.yml file

![ENV VARS NGINX](https://user-images.githubusercontent.com/114196715/206769355-2f7d6bd9-8e68-4ef6-8d95-4948afc43192.png)
      
* Run the playbook

      
## CHALLENGE

After running the playbook, the following error was encountered. The error was found to be due to the following reasons;

![nginx error](https://user-images.githubusercontent.com/114196715/206769477-b066b762-35f2-4763-8233-ffcd8c2699f2.png)
      
Reason 1:  It was discovered that we can not have apache and nginx listening at port 80 simultaneously. As a result of this, the nginx service failed to install. I had to disable and stop apache in defaults/main.yml file of apache role.

Reason 2: My nginx vhost configuration was found to be missing quite an important snippet: `marker: "# {mark} ANSIBLE MANAGED BLOCK {{ item.name }}"`.

![initial snippet](https://user-images.githubusercontent.com/114196715/206769603-464e522d-e9c0-4068-a7df-5ff2a8f35bc7.png)
      
As a result, the blockinfile didn't loop through the two web servers that was meant to be inserted in the "/etc/hosts" file. Only one web server was inserted as shown below from the `nginx -t` configuration test output:

![nginx -t ](https://user-images.githubusercontent.com/114196715/206769770-610e0052-82ed-490f-9d9a-4a02c02f3c1d.png)
      
Solution: The missing snippet was included;

![snippet updated](https://user-images.githubusercontent.com/114196715/206769937-5efb0b04-735c-4f21-b3dc-084afa50c9ff.png)
      
** The playbook was run and nginx was configured successfully.

![nginx lb play 1](https://user-images.githubusercontent.com/114196715/206770187-17d96461-bb1a-451e-83de-3045b4e1bbcc.png)

![nginx lb play 2](https://user-images.githubusercontent.com/114196715/206770189-7598eb5e-32ea-431b-a2b8-27a0ff2b4945.png)

![nginx lb play 3](https://user-images.githubusercontent.com/114196715/206770178-b37cb590-e060-499a-aa72-b16f01af2fd9.png)

* Input the public IP of the loadbalancer in a web browser;

![nginx final lb site](https://user-images.githubusercontent.com/114196715/206767757-d231c182-6bc1-4f4a-9f16-6cf358079149.png)

-----------END------------------
