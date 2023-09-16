# Project-Base-Learning-13
ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

## PROJECT TASK

- In this project we will introduce **dynamic assignments** by using Ansible **include** module. Our target is to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.

## BACKGROUND KNOWLEDGE

**Ansible** is a popular open-source configuration management and automation tool that is used for various IT tasks such as *configuration management*, *application deployment*, and *orchestration*. One of the key features of Ansible is its ability to use variables to store and manipulate data. These variables can be defined in various ways and can be used to perform *dynamic assignments* in ansible playbooks.

**Dynamic assignments** in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments. This project will focus on use of dynamic assigments

**Static Assignments** here, the values of variables are defined and fixed at the time of writing the playbook. 

There are several ways to perform dynamic assignments in ansible. Some of the common methods are:

- **Using conditionals** : Conditionals in ansible allow you to specify a block of tasks that should be executed only if a certain condition is met. For example, you can use the when clause to specify a condition under which a task should be executed.

- **Using loops** : Loops in ansible allow you to repeat a set of tasks for a given number of times or for each item in a list. You can use loops to perform dynamic assignments by looping over a list of items and assigning a value to a variable for each iteration.

- **Using set_fact module** : The set_fact module in ansible allows you to set the value of a variable based on the output of a command or expression. You can use this module to perform dynamic assignments by specifying a command or expression that returns the value you want to assign to the variable.

- **Using register module** : The register module in ansible allows you to store the output of a task in a variable. You can use this module to perform dynamic assignments by storing the output of a task in a variable and using it later in the playbook.

When the **import module** is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when **include module** is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

In most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic assignments, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

## Setup and technologies used in Project 13

- Two RHEL8 Web Servers
- One RHEL8 DB Server
- One RHEL8 NFS server
- One LB server(based on Ubuntu 20.04 LTS)
- One **Jenkins/Ansible Server** server(based on Ubuntu 20.04 LTS)
- Two 'UAT' Web Server

- Your target architecture will look like this:

![12_1](https://github.com/EzeOnoky/Project-Base-Learning-12/assets/122687798/300339c9-aace-47a4-9497-4fe1f1184c74)

## STEP 1 - INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE

On AWS, Start your Ansible-Jenkins server(from previous project 12). SSH into the server using VS Code IDE. In your https://github.com//ansible-config-mgt GitHub repository, start a new branch and call it **dynamic-assignments**.

```
git status
git pull
git checkout -b dynamic-assignment
```

Then while on the new branch, proceed to create a new directory and name it **dynamic-assignments**. While inside this directory, create a new file **env-vars.yml**. We will instruct **site.yml** to **include** this playbook later. For now, let us keep building up the structure.

We will be using the same Ansible to configure multiple environments and each of these environments will have certain unique attributes, such as servername and ip-address etc. We will set values to variables per specific environment.

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as **servername, ip-address** etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder **env-vars**, then for each environment, create new **YAML** files which we will use to set variables.

Below is how my layout  now look like...

# 13_2

Now paste the instruction below into the **env-vars.yml** file.

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

# 13_3 shwoing above

N/B: We used **include_vars** module instead of **include**. This is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include*_ must be used. These are:

- include_role
- include_tasks
- include_vars

In the same version, variants of import were also introduced such as:

- import_role
- import_tasks

We made use of a special variables **{ playbook_dir }** and **{ inventory_file }**. 

**{ playbook_dir }** will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem.

**{ inventory_file }** on the other hand will dynamically resolve to the name of the inventory file being used, then append **.yml** so that it picks up the required file within the **env-vars** folder. 

We are including the variables using a loop. 

**with_first_found** implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

## STEP 2 - UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS

Update site.yml file to make use of the dynamic assignment with the snippet below:

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

Above Explained...

### 2A   - Download Mysql Ansible Role from Community Roles

Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. To avoid manually doing all these, we can download and install open-source roles. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

The community roles can be found [here](https://galaxy.ansible.com/home), for this project, we will be using MySQL role developed by [geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql)

To preserve your GitHub in actual state after you install a new role – make a commit and push to master - ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose.

Using below, We have to install git on Jenkins-Ansible server and then configure Visual Studio Code to work with this directory. Below tragets to create a new branch - **roles-feature** while connected to **ansible-config-mgt** directory

```
git init
git --version
git pull https://github.com/EzeOnoky/ansible-config-mgt
git remote add origin https://github.com/EzeOnoky/ansible-config-mgt
git branch roles-feature
git switch roles-feature
git status
```

# 13_2 success execution of above

So for now, Jenkins jobs and webhook will no longer be needed in this project.

Before we continue the download of from the Ansible galaxy server, confirm you are on the new `roles-feature` branch, by running `git status`

cd into roles directory, 
Create the Mysql role(recall in project 12, `ansible-galaxy init webserver` was used to create a role - webserver ...with the init command)
Rename the folder(geerlingguy.mysql) to mysql

```
git status
cd roles
ls
ansible-galaxy install geerlingguy.mysql
mv geerlinguy.mysql/ mysql
ls
```

# 13_3 success execution of above

Read **README.md** file and edit roles configuration to use correct credentials for MySQL required for the tooling website, Edit the **defaults/main.yml** in the **mysql** role

# 13_4 refer to kebe/solo pictures

Create a **db.yml** file in the **static assignment** that will point to the **mysql** role

# 13_5 refer to Solo pictures

Now it is time to upload the changes into your GitHub

```
git status
git add .
git commit -m "updated geerlinguy mysql role"
git push --set-upstream origin roles-feature
```

Make a commit and push the roles-feature branch and merge to main branch.

# 13_6 refer to Solo pictures

On the GITHUB repo, Create a **pull request** and merge to the **main** branch if we are satisfied with your codes.


## STEP 3 -  LOAD BALANCER ROLES

### 3A   - Download NGINX & APACHE Community Roles from Ansible Galaxy Server

We want to be able to choose which Load Balancer to use, **Nginx** or **Apache**, so we need to have two roles respectively:

- Nginx
- Apache

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – *this is where you can make use of* **variables**

To proceed in creating the Ansible **role**, we can decide to either develop our own roles, or find available roles from the Ansible Galaxy community. To make life easy, we will create **roles** for *Nginx* and *Apache* from the community roles, as earlier mentioned, we will download ready to use community roles from **geerlingguy** on Ansible Galaxy Server.

Check [here](https://galaxy.ansible.com/geerlingguy) , search and Copy the link for the *Nginx* and *Apache* roles. 

# 13_7 Showing geerlingguy site - refer solo pix

Ensure you create the roles in the **roles** directory...Rename the newly download roles to **Nginx** and **Apache** respectively.

```
cd roles
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.apache
mv geerlingguy.nginx/ nginx
mv geerlingguy.apache/ apache
```

# 13_7 Showing successful execution - refer solo pix

### 3B - Introduce Variables, Update static-assignment and site.yml files to refer the downloaded roles

- 3B_1 : Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of **variables**

- 3B_2 : Inside the new Nginx and Apache roles, declare a variable in **defaults/main.yml** file . Name each variables **enable_nginx_lb** and **enable_apache_lb** respectively.

- 3B_3 : Set both values to false like this `enable_nginx_lb: false` and `enable_apache_lb: false`

- 3B_4 : Declare another variable in both roles `load_balancer_is_required` and set its value to false as well

- 3B_5 : Update both assignment and site.yml files respectively


**3B_2, 3B_3, 3B_4, 3B_5 executed**
```
enable_nginx_lb: false
load_balancer_is_required: false

enable_apache_lb: false
load_balancer_is_required: false
```

# 13_8 Showing successful execution for both NGINX & APACHE - refer solo pix


Also edit the defaults/main.yml files to capture private IP address of WEB1 & WEB2

# 13_9 - refer solo/KEBE pix to understand

Now we proceed to update both `static-assignments` directory with a new file - **loadbalancers.yml** and also update the `site.yml` files respectively

In the **static assignments** directory, create a file **loadbalancers.yml** file, Paste the snippet below in the **loadbalancers.yml** file

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

# 13_10 - pix showing above is done, refer solo pix to understand

As seen above, with the `when` condition, It is used to decide the nature of load balancer installed in an environment depending on which variables is set to true. Recall the variables introduced in **3B_2** above.

Then update the **playbooks/sites.yml** file with below snippet

```
- name: Loadbalancers assignment
    - import_playbook: ../static-assignments/loadbalancers.yml
      when: load_balancer_is_required
```

# 13_11 - pix showing above is done, refer solo/kebe pix to understand

Now you can make use of **env-vars\uat.yml** file to define which loadbalancer to use in UAT environment by setting respective environmental variable to `true`

You will activate load balancer, and enable nginx by setting these in the respective environment’s `env-vars` file.

```
enable_nginx_lb: true
load_balancer_is_required: true
```

# 13_12 - pix showing above is done, refer Kebe pix to understand

The same must work with apache LB, so you can switch it by setting respective environmental variable to **true** and other to **false**

Now when the nginx server is set to run, it skips all the plays on apache server and vice versa. We are using the same port address, port 80, so remember to stop the service of either nginx when you want to run apache load balancer and vice versa.


Then we update the **inventory/uat.yml**

# 13_13 - pix showing above is done, refer solo pix to understand

Check if the ansible can access the inventory

`ansible all -m ping -i inventory/uat.yml`

# 13_13 - pix showing above is done, refer solo pix to understand


Push the **dynamic assignment** branch and merge

```
git status
git add .
git comit -m "add env-vars,dynamic assignments and update site.yml"
git push origin dynamic-assignment
```

# 13_14 - pix showing above is done, refer solo pix to understand

Creat a pull request and then merge

Now we can run the Playbook ...skipping apache

`ansible-playbook playbooks/site.yml -i inventory/uat.yml`

# 13_15 - pix showing above is done, 

Notice how it skips all apache2 plays, this is because nginx load balancer is activated.


Now we can run the Playbook ...skipping nginx

`ansible-playbook playbooks/site.yml -i inventory/uat.yml`

# 13_16 - pix showing above is done, 

Notice how it skips all nginx plays, this is because apache2 load balancer is activated.





In both scenarios, the web server was reachable on same port because i always stopped the service first on the play before i switched to a new service

# Congratulation EZE, you have -----
