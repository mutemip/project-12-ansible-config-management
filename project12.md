# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project we will continue working with ansible-config-mgt repository and make some improvements of our code.

*Code Refactoring*
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

## Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require [Copy Artifact](https://plugins.jenkins.io/copyartifact/) plugin.

1. Go to our `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact` – we will store there all artifacts after each build.
   ```
   sudo mkdir ansible-config-artifact
   ```

2. Change permissions to this directory, so Jenkins could save files there
   ```
   chmod -R 0777 ansible-config-artifact
   ```

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for `Copy Artifact` and install this plugin without restarting Jenkins


![p12-1](https://user-images.githubusercontent.com/64135078/206047163-f7df19bb-7d70-4ebc-9ffc-93cf1944d9aa.png)

4. Create a new Freestyle project (as done in [project9]) and name it `save_artifacts`.

5. This project will be triggered by completion of our existing ansible project. Configure it accordingly:

![p12-2](https://user-images.githubusercontent.com/64135078/206048159-4183fd73-5426-4a67-b432-78b5fa3af6b0.png)
![p12-3](https://user-images.githubusercontent.com/64135078/206048188-ef555154-3b84-4008-8da8-bc0c1749dda3.png)

*Note:* we can configure number of builds to keep in order to save space on the server, for example, 
we might want to keep only last 2 or 5 build results. we can also make this change to our `ansible` job.

6. The main idea of save_artifacts project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. 
To achieve this, create a Build step and choose Copy artifacts from other project`, 
specify `ansible` as a source project and /`home/ubuntu/ansible-config-artifact` as a target directory.

![p12-4](https://user-images.githubusercontent.com/64135078/206048675-45f71079-f86c-471f-8110-444892f73b99.png)

![p12-5](https://user-images.githubusercontent.com/64135078/206048707-e6c2b650-d57a-445a-b847-db55ba16c71d.png)

7. Test our set up by making some change in README.MD file inside our `ansible-config-mgt` repository (right inside `master` branch).

If both Jenkins jobs have completed one after another – we shall see our files inside /`home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to our `master` branch.

![screen](https://user-images.githubusercontent.com/64135078/206048863-fec34530-a616-462f-99d2-ebc4a26912ba.png)

## Step 2 - Refactor Ansible code by importing other playbooks into `site.yml`
Before starting to refactor the codes, ensure that we have pulled down the latest code from `master` (main) branch, and created a new branch, name it `refactor`.

DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used, but we always have an answer to question "why?". Why do we need to change something if it works well?

In [Project 11]) we wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine we have many more tasks and we need to apply this playbook to other servers with different requirements. In this case, we will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that we need to add for certain server/OS families. Very fast it will become a tedious exercise and our playbook will become messy with many commented parts. our DevOps colleagues will not appreciate such organization of our codes and it will be difficult for them to use our playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let's see code re-use in action by importing other playbooks.

1. Within `playbooks` folder, create a new file and name it `site.yml` – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that we created previously.

2. Create a new folder in root of the repository and name it `static-assignments`. The *static-assignments* folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

3. Move `common.yml` file into the newly created `static-assignments` folder.
   
4. Inside `site.yml` file, import `common.yml` playbook:
   
   ```
   ---
   - hosts: all
   - import_playbook: ../static-assignments/common.yml
   ```
![p12-6](https://user-images.githubusercontent.com/64135078/206049376-4bc4b3bd-9ae4-4917-b521-a01c56d0858e.png)

The code above uses built in [import_playbook](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_playbook_module.html)Ansible module.
Your folder structure should look like this;

![screen](https://user-images.githubusercontent.com/64135078/206049675-56796c9d-29b1-46a9-a4ae-e5edc1504faf.png)

5. Run `ansible-playbook` command against the dev environment

   Since you need to apply some tasks to your `dev` servers and `wireshark` is already installed – you can go ahead and create another playbook under `static-assignments` and name it `common-del.yml`. In this playbook, configure deletion of `wireshark` utility.

   ```
   ---
   - name: update web, nfs and db servers
   hosts: webservers, nfs, db
   remote_user: ec2-user
   become: yes
   become_user: root
   tasks:
   - name: delete wireshark
      yum:
         name: wireshark
         state: removed

   - name: update LB server
   hosts: lb
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

![p12-8](https://user-images.githubusercontent.com/64135078/206049965-48bb60f2-3763-4106-8455-022510912777.png)

update `site.yml` with `- import_playbook: ../static-assignments/common-del.yml` instead of `common.yml`

![p12-9](https://user-images.githubusercontent.com/64135078/206050013-d3047b63-2625-41b8-a2da-8045acbc2743.png)

run it against dev servers:

   ```
   cd /home/ubuntu/ansible-config-mgt/

   ansible-playbook -i inventory/dev.yml playbooks/site.yml
   ```


![p12-10](https://user-images.githubusercontent.com/64135078/206050207-c8bb5f1a-3bf2-466d-bdac-4ab2af4e0ad8.png)

Make sure that wireshark is deleted on all the servers by running wireshark --version

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

## Step 3 – Configure UAT Webservers with a role `Webserver`

We have our nice and clean `dev` environment, so let us put it aside and configure 2 new Web Servers as `uat`. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated `role` to make our configuration reusable.

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – `Web1-UAT` and `Web2-UAT`.

2. To create a role, you must create a directory called `roles/`, relative to the playbook file or in `/etc/ansible/` directory.
   
   There are two ways how you can create this folder structure:

   - Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)
   ```
   mkdir roles
   cd roles
   ansible-galaxy init webserver
   ```

   - Create the directory/files structure manually

*Note:* You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on `Jenkins-Ansible` server.

The entire folder structure should look like below, but if you create it manually – you can skip creating `tests`, `files`, and `vars` or remove them if you used `ansible-galaxy`


![p12-11](https://user-images.githubusercontent.com/64135078/206050843-c715049a-3a1f-44d9-b941-fe7fc83e25c9.png)

After removing unnecessary directories and files, the roles structure should look like this

![p12-12](https://user-images.githubusercontent.com/64135078/206050900-5768c5b1-20de-46f9-b5f1-a69e7601039f.png)


3. Update your inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of your 2 UAT Web servers
   
   ```
   [uat-webservers]
   <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

   <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
   ```
   
   ![p12-13](https://user-images.githubusercontent.com/64135078/206051250-b6bcd72b-57cd-4210-94c1-0ab2c59ae467.png)
   
  4. In `/etc/ansible/ansible.cfg` file uncomment roles_path string and provide a full path to your roles directory 
  5. `roles_path    =/home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.

![p12-14](https://user-images.githubusercontent.com/64135078/206051349-f9c96074-b715-45b2-8d0b-7c58b3ebeb48.png)

5. It is time to start adding some logic to the webserver role. Go into `tasks` directory, and within the `main.yml` file, start writing configuration tasks to do the following:
   - Install and configure Apache (`httpd` service)
   - Clone Tooling website from GitHub `https://github.com/<your-name>/tooling.git`.
   - Ensure the tooling website code is deployed to `/var/www/html` on each of 2 UAT Web servers.
   - Make sure `httpd` service is started

Your `main.yml` may consist of following tasks:
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent

```

## Step 4 – Reference `Webserver` role
Within the `static-assignments` folder, create a new assignment for *uat-webservers* `uat-webservers.yml`. This is where you will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`.

So, we should have this in `site.yml`

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

## Step 5 – Commit & Test
Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-artifacts/ directory.

Now run the playbook against your uat inventory and see what happens:

```
ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml
```

![p12-15](https://user-images.githubusercontent.com/64135078/206051553-3bdd9c5b-4745-4485-84cd-0423be347105.png)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

*Tip* - Do remember to enable HTTP connection in your ec2 instances

http://Web1-UAT-Server-Public-IP-or-Public-DNS-Name/index.php

![p12-16](https://user-images.githubusercontent.com/64135078/206051777-c86544c8-814b-494c-b535-b7cab5c3bc95.png)

