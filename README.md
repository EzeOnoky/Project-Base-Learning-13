# Project-Base-Learning-13
ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

## PROJECT TASK

- In this project we will introduce **dynamic assignments** by using Ansible **include** module. Our target is to use Ansible configuration management tool to prepare UAT environment for Tooling web solution. Last 2 projects have already equipped you with some knowledge and skills on Ansible, so you can perform configurations using `playbooks` , `roles` and `imports`. Now you will continue configuring your UAT servers, learning and practicing new Ansible concepts and modules.

## BACKGROUND KNOWLEDGE

**Ansible** is a popular open-source configuration management and automation tool that is used for various IT tasks such as *configuration management*, *application deployment*, and *orchestration*. One of the key features of Ansible is its ability to use variables to store and manipulate data. These variables can be defined in various ways and can be used to perform *dynamic assignments* in ansible playbooks. Visit [Ansible Documentation](https://docs.ansible.com/) page to learn  latest updates on modules and their usage.


**Dynamic assignments** in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments. This project will focus on use of dynamic assigments

**Static Assignments** here, the values of variables are defined and fixed at the time of writing the playbook. 

There are several ways to perform dynamic assignments in ansible. Some of the common methods are:

- **Using conditionals** : Conditionals in ansible allows you to specify a block of tasks that should be executed only if a certain condition is met. For example, you can use the when clause to specify a condition under which a task should be executed.

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


Then while on the new branch, proceed to create a new directory and name it **dynamic-assignments**. While inside this directory, create a new file **env-vars.yml**. We will instruct **site.yml** to **include** this playbook later. For now, let us keep building up the structure.
```
git status
git pull
git checkout -b dynamic-assignments
mkdir dynamic-assignments
cd dynamic-assignments/
touch env-vars.yml
```
![13_1](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/3c51c29d-e744-4ae6-bd0c-fe24a7059d35)


![13_2](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/2b791509-c3c9-487a-94ea-b2fed6a675bf)


We will be using the same Ansible to configure multiple environments and each of these environments will have certain unique attributes, such as servername and ip-address etc. We will set values to variables per specific environment.

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as **servername, ip-address** etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder **env-vars**, then for each environment, create new **YAML** files which we will use to set variables.

```
mkdir env-vars
cd env-vars
touch dev.yml prod.yml stage.yml uat.yml
```
![13_2a](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/82866ce5-625a-494f-9fd0-53fb05d641a9)


Below is how my layout  now look like after excution of above...

![13_3](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/80870f77-d767-4315-b63a-6d2d49277b89)

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

![13_4](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/03b2558c-2c94-443b-8918-493f915236b0)


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

Now you need to commit all the recent changes done while on branch `dynamic-assignments` , push it to github repo - ansible-config-mgt, and merge it to the main branch.

```
git status
git add .
git commit -m "dynamic-assignments branch updates"
git push origin dynamic-assignments
```

![13_5](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/8e212484-8a42-405c-8b76-2f98dedcaa7c)


Merging to the main branch is confirmed OK..

![13_6](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/59058e5c-e4f3-4621-b661-f0d6cac78f6c)


### 2A   - Download Mysql Ansible Role from Community Roles

Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. To avoid manually doing all these, we can download and install open-source roles. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

The community roles can be found [here](https://galaxy.ansible.com/home), for this project, we will be using MySQL role developed by [geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql)

To preserve your GitHub in actual state after you install a new role – make a commit and push to master - ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose.

Using below, We have to install git on Jenkins-Ansible server and then configure Visual Studio Code to work with this directory. Below tragets to create a new branch - **roles-feature** while connected to **ansible-config-mgt** directory

Now return to your Jenkin-Ansible SSH logon,

```
git init
git --version
git pull
git status
git branch roles-feature
git switch roles-feature
git status
```

Compare 1 & 2 below, and you will see that after the `git pull` ....the newly created folders in Step 1 & 2 are now updated on the Jenkins-Ansible server.


![13_7](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/9aa314cd-a020-4e47-9329-550f40f505ae)


![13_8](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/79c7b088-cdad-4157-ac69-07f382f486aa)


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
mkdir mysql
mv geerlinguy.mysql/ mysql
ls
```

![13_9](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/ae4788ec-0f1e-4320-87c3-7df1f8adcb79)


Read **README.md** file and edit roles configuration to use correct credentials for MySQL required for the tooling website, Edit the **defaults/main.yml** in the **mysql** role

![13_10](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/322022ab-2e99-4af5-bb9c-c18f61978e55)


Create a **db.yml** file in the **static assignment** that will point to the **mysql** role

# 13_5 refer to Solo pictures

Now it is time to upload the changes into your GitHub

```
git status
git add .
git commit -m "updated geerlinguy mysql role"
git push --set-upstream origin roles-feature
```

![13_11](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/232f076c-b149-4ae1-9b22-f601834d06ae)


Make a commit and push the roles-feature branch and merge to main branch.

On the GITHUB repo, Create a **pull request** and merge to the **main** branch if we are satisfied with your codes.

![13_12](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/650fd2a3-aa6d-4c8e-877f-f96c185fd031)



## STEP 3 -  LOAD BALANCER ROLES

### 3A   - Download NGINX & APACHE Community Roles from Ansible Galaxy Server

We want to be able to choose which Load Balancer to use, **Nginx** or **Apache**, so we need to have two roles respectively:

- Nginx
- Apache

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – *this is where you can make use of* **variables**

To proceed in creating the Ansible **role**, we can decide to either develop our own roles, or find available roles from the Ansible Galaxy community. To make life easy, we will create **roles** for *Nginx* and *Apache* from the community roles, as earlier mentioned, we will download ready to use community roles from **geerlingguy** on Ansible Galaxy Server.

Check [here](https://galaxy.ansible.com/geerlingguy) , search and Copy the link for the *Nginx* and *Apache* roles. 


Ensure you create the roles in the **roles** directory...Rename the newly download roles to **Nginx** and **Apache** respectively.

```
cd roles
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.apache
mv geerlingguy.nginx/ nginx
mv geerlingguy.apache/ apache
```

![13_13](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/7f1d7700-08cb-4419-9676-1513ae803810)


### 3B - Introduce Variables, Update static-assignment and site.yml files to refer the downloaded roles

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of **variables**

- 3B_1 : Inside the new Nginx and Apache roles, declare a variable in **defaults/main.yml** file . Name each variables **enable_nginx_lb** and **enable_apache_lb** respectively. Set both values to false like this

```
enable_nginx_lb: false
enable_apache_lb: false
```

- 3B_2 : Declare another variable in both roles `load_balancer_is_required` and set its value to false as well

```
enable_nginx_lb: false
load_balancer_is_required: false

enable_apache_lb: false
load_balancer_is_required: false
```


Also edit the defaults/main.yml files to capture private IP address of WEB1 & WEB2, check your jenkins-ansible server `cat /etc/hosts` for the list

![13_14](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/5e7fb671-d5c8-4b3f-8f48-d7c4f0592270)

Above is for Nginx roles - **defaults/main.yml**, NOTE **nginx_extra_http_options:** which was previous commented out, was uncommented.


![13_15](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/d0f12c33-c8b7-447a-b708-73a05aedd10b)

Above is for Nginx roles - **defaults/main.yml**


- 3B_5 : Update both static assignment and site.yml files respectively

Now we proceed to update both `static-assignments` directory with a new file - **loadbalancers.yml** and also update the `site.yml` files respectively

In the **static assignments** directory, create a file **loadbalancers.yml** file, Paste the snippet below in the **loadbalancers.yml** file

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

As seen above, with the `when` condition, It is used to decide the nature of load balancer installed in an environment depending on which variables is set to true. Recall the variables introduced in **3B_2** above.

Then update the **playbooks/sites.yml** file with below snippet

```
- name: Loadbalancers assignment
    - import_playbook: ../static-assignments/loadbalancers.yml
      when: load_balancer_is_required
```

![13_16](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/39b8b022-ded2-4fec-b9f9-4f497998ab82)


Now you can make use of **env-vars\uat.yml** file to define which loadbalancer to use in UAT environment by setting respective environmental variable to `true`

You will activate load balancer, and enable nginx by setting these in the respective environment’s `env-vars` file.

```
enable_nginx_lb: true
load_balancer_is_required: true
```

The same must work with apache LB, so you can switch it by setting respective environmental variable to **true** and other to **false**

![13_17](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/2d220b72-b324-4e1d-b879-bd58b6276e46)


Now when the nginx server is set to run, it skips all the plays on apache server and vice versa. We are using the same port address, port 80, so remember to stop the service of either nginx when you want to run apache load balancer and vice versa.

Then we update the **inventory/uat.yml**

![13_18](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/bdfe38f0-b068-4a81-8655-dd57f30e0c6c)

```
ssh ec2-user@172.31.41.85
ssh ec2-user@172.31.37.3
ssh ec2-user@172.31.37.56
ssh ubuntu@172.31.34.225
```

Check if the ansible can access the inventory

`ansible all -m ping -i inventory/uat.yml`

![13_19](https://github.com/EzeOnoky/Project-Base-Learning-13/assets/122687798/190097a6-af47-4e12-aafc-39915bb5a2c4)


Push the **roles-feature** branch and merge to the main branch

```
git status
git add .
git commit -m "final changes on role-features branch 1"
git push origin roles-feature
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


