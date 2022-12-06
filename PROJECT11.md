# INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

## Step 1

1. Update `Name` tag on your `Jenkins` EC2 Instance to `Jenkins-Ansible`. We will use this server to run playbooks.
![p11](https://user-images.githubusercontent.com/64135078/206034048-303e7c81-cccb-47af-9992-bd680aafb32b.png)

2. In your GitHub account create a new repository and name it `ansible-config-mgt`
3. Instal Ansible
   ```
   sudo apt update
   sudo apt install ansible
   ```

   Check your Ansible version by running 
   ```
   ansible --version
   
