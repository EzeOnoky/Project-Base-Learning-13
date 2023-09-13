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

Create a new directory and name it **dynamic-assignments**. Then inside this directory, create a new file **env-vars.yml**. We will instruct **site.yml** to **include** this playbook later. For now, let us keep building up the structure.

```
git status
git pull
git checkout -b dynamic-assignment
```

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

### 2A   - COMMUNITY ROLES
