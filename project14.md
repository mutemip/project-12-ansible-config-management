# CONTINOUS INTEGRATION WITH JENKINS, ANSIBLE, ARTIFACTORY SONARQUBE AND PHP

## Introduction
In this project, the concept of CI/CD is implemented whereby php application from github are pushed to Jenkins to run a multi-branch pipeline job(build job is run on each branches of a repository simultaneously) which is better viewed from Blue Ocean plugin. This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job is packaged and pushed to sonarqube server for testing before it is deployed to artifactory from which ansible job is triggered to deploy the application to production environment.

The following are the steps taken to achieve this:

# STEP - 0: Setting Up Servers
I launched 3 EC2 Instances, one for Jenkins server(t2.medium), another for MySQL database(RedHat) and another is used for SonarQube Server.

# STEP - 1: Configuring Ansible For Jenkins Deployment
In order to run ansible commands from Jenkins UI, the following outlines the steps taken:
(Note: I already have jenkins installed on the server, click [here](https://www.jenkins.io/doc/book/installing/)to see documentation on how to install jenkins)

- Installing Blue Ocean plugin from ‘manage plugins’ on Jenkins:
  On the dashboard, go to `Manage Jenkins`>`Manage Plugins`>`Available`, then search `Blue Ocean` on the search box.
  Select `Blue Ocean` from the search results, then click `Install without restart`.
  
  ![p14-1](https://user-images.githubusercontent.com/64135078/206053608-b5e1d1cf-a001-4f23-a08f-86a56c5255a5.png)

  ![p14-2](https://user-images.githubusercontent.com/64135078/206053634-8e9a2ce3-8857-4be4-b780-7c411c8bf157.png)

- Creating new pipeline job on the Blue Ocean UI from Github
  Go to the Dashboard, on the left side pane, select `Blue Ocean`

![p14-3](https://user-images.githubusercontent.com/64135078/206053814-c882a51b-f8d7-4daa-8298-1bce2eefe753.png)

![p14-4](https://user-images.githubusercontent.com/64135078/206053820-7876c02a-7512-42b6-a9b1-eff473ba78f5.png)

Select `Create a new Pipeline` > `GitHub` > Enter an access token to github repo > click `connect` > `Select your preferred organization` > Then select the repo - `ansible-config-mgt` > click `Create Pipeline`

![p14-5](https://user-images.githubusercontent.com/64135078/206054017-b18bb93c-0512-4dc1-8e32-6bbbea8409c7.png)

- Creating a directory in the root of ansible-config-mgt directory called deploy, and creating a file in it called `Jenkinsfile`.
  Add the code snippet below to start building the `Jenkinsfile` gradually. This pipeline currently has just one stage called `Build` and the only thing we are doing is using the shell script module to echo Building Stage
  ```
  pipeline {
    agent any

    stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }
      }
  }
  ```

- Now go back into the Ansible pipeline in Jenkins, and select `configure`

![p14-6](https://user-images.githubusercontent.com/64135078/206054299-4a551153-1bf3-4c8a-9454-5e2ed82b11e4.png)

 Scroll down to `Build Configuration` section and specify the location of the Jenkinsfile at `deploy/Jenkinsfile`
 
![p14-7](https://user-images.githubusercontent.com/64135078/206054428-7d665a08-b6bf-4b16-b53e-139d74dde9b5.png)

Click `Apply` and `Save`.

  Back to the pipeline again, this time click "Build now"


This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

![p14-8](https://user-images.githubusercontent.com/64135078/206055018-2ea3f922-d131-4b44-8233-978b64907c34.png)

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

1. Click on Blue Ocean
2. Select your project
3. Click on the play button against the branch
4. 
![p14-9](https://user-images.githubusercontent.com/64135078/206055673-4c0d8983-9931-4935-bb9c-1468053c43ed.png)

Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.
1. Create a new git branch and name it `feature/jenkinspipeline-stages`
2. Currently we only have the `Build` stage. Let us add another stage called `Test`. Paste the code snippet below and push the new changes to GitHub.
```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```
3. To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

-  Click on the "Administration" button
- Navigate to the Ansible project and click on "Scan repository now"
- Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.
- In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

![p14-10](https://user-images.githubusercontent.com/64135078/206056326-348f16c8-ed51-4b67-806c-1d984adcc17f.png)
![p14-11](https://user-images.githubusercontent.com/64135078/206056353-e3c1414c-5853-4992-af58-9bbe801ab6b4.png)

```
A QUICK TASK FOR YOU!
1. Create a pull request to merge the latest code into the main branch
2. After merging the PR, go back into your terminal and switch into the main branch.
3. Pull the latest change.
4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
   1. Package 
   2. Deploy 
   3. Clean up
5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
6. Eventually, your main branch should have a successful pipeline like this in blue ocean
```
![](images/quick-task-1.png)


## Setting up Ansible on Jenkins
Make sure to structure your Ansible inventory to look like the one below:

```
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

ci inventory file
```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>

```

dev Inventory file
```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

pentest inventory file
```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

1. Installing Ansible on Jenkins server
   ```
   sudo dnf install -y ansible-core
   ansible --version
   ```
   For this project, there are some dependencies we need for the ansible to work fine. Make sure to run the commands below:
   ```
   sudo yum install python3 python3-pip wget unzip git -y
   sudo python3 -m pip install --upgrade setuptools
   sudo python3 -m pip install --upgrade pip
   sudo python3 -m pip install PyMySQL
   sudo python3 -m pip install mysql-connector-python
   sudo python3 -m pip install psycopg2==2.7.5 --ignore-installed

   # For postgres database
   sudo ansible-galaxy collection install community.postgresql
   ```

2. Installing Ansible plugin in Jenkins UI
   On the Availables tab, search - `ansible`


![p14-12](https://user-images.githubusercontent.com/64135078/206056552-20d49679-f1c5-4b71-8b4f-f2881b64e51b.png)

select, and install the plugin.


3. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)
   ```
   pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
      }

      stages {
        stage('Initial cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps{
                git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/franklinobasy/ansible-config-mgt.git'
            }
          }

        stage('Prepare Ansible For Execution') {
            steps {
              sh 'echo ${WORKSPACE}' 
              sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
            }
          }

        stage('Run Ansible playbook') {
            steps {
              ansiblePlaybook become: true, credentialsId: 'ec2-instances-private-key', disableHostKeyChecking: true, installation: 'Ansible', inventory: 'inventory/dev', playbook: 'playbooks/site.yml'
            }
          }
        
        stage('Clean Workspace after build'){
            steps{
              cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
          }
      }
    }
   ```

**Posible errors to look out for**
- Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master 
  
- Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the `.ansible.cfg` file alongside `Jenkinsfile` in the `deploy` directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.
  
 /deploy/ansible.cfg
  ```
  [defaults]
  timeout = 160
  callback_whitelist = profile_tasks
  log_path=~/ansible.log
  host_key_checking = False
  gathering = smart
  ansible_python_interpreter=/usr/bin/python3
  allow_world_readable_tmpfiles=true


  [ssh_connection]
  ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes

  ```

  ** Possible issues to watch out for when you implement this **
  - Remember that `ansible.cfg` must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor `sed` to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.

  - If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.

  - Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.

![p14-13](https://user-images.githubusercontent.com/64135078/206056898-551a09d6-b593-4a79-a038-8f469c55149f.png)

If everything goes well for you, it means, the Dev environment has an up-to-date configuration. But what if we need to deploy to other environments?

Are we going to manually update the Jenkinsfile to point inventory to those environments? such as sit, uat, pentest, etc.
Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.
Think about those for a minute and try to work out which one sounds more like a better solution.

Manually updating the Jenkinsfile is definitely not an option. And that should be obvious to you at this point. Because we try to automate things as much as possible.

Well, unfortunately, we will not be doing any of the highlighted options. What we will be doing is to parameterise the deployment. So that at the point of execution, the appropriate values are applied.


### Parameterizing `Jenkinsfile` For Ansible Deployment
To deploy to other environments, we will need to use parameters.
1. Update `Jenkinsfile` to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.
```
 parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
```
2. In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}
Your jenkinsfile file should finally look like this:
```
pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }
  
  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
   }

  stages {
    stage('Initial cleanup') {
        steps {
            dir("${WORKSPACE}") {
                deleteDir()
            }
        }
    }

    stage('Checkout SCM') {
        steps{
            git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/franklinobasy/ansible-config-mgt.git'
        }
      }

    stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
      }

    stage('Run Ansible playbook') {
        steps {
          ansiblePlaybook become: true, credentialsId: 'ec2-instances-private-key', disableHostKeyChecking: true, installation: 'Ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml'
        }
      }
    
    stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
  }
}
```

After pushing to feature/jenkinspipeline-stages: You should see this as the new configurations:
![p14-14](https://user-images.githubusercontent.com/64135078/206057072-8c16b2d2-36f7-4bab-bfdb-b6fa39e7f6e5.png)

# STEP 2 - CI/CD PIPELINE FOR TODO APPLICATION
We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

- Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part).

## Phase 1 – Prepare Jenkins
1. Fork the repository below into your GitHub account
   `https://github.com/darey-devops/php-todo.git`

2. On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)
   ```
    yum module reset php -y
    yum module enable php:remi-7.4 -y
    yum install -y php  php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd    php-fpm php-json
    systemctl start php-fpm
    systemctl enable php-fpm
   ```

   Install composer
   ```
   curl -sS https://getcomposer.org/installer | php 
   sudo mv composer.phar /usr/bin/composer
   composer --version
   ```

3. Install Jenkins plugins
  - Plot plugin
    We will use `plot` plugin to display tests reports, and code coverage information.
  - Artifactory plugin
    The `Artifactory` plugin will be used to easily upload code artifacts into an Artifactory server.

4. Spin up another server that will host the jfrog artifactory
   Learn how to installk artifactory [here](https://jfrog.com/open-source/)
   Note that we will require a minimum intance type of `t2-medium`, also enable TCP port `8081` and `8082` in the security group. This is the port used by artifactory

5. On the jenkins server, inside the `ansible-config-mgt` folder, download `Artifactory` setup roles to the roles path
  Note: set roles_path to `~/ansible-config-mgt/roles`

![p14-15](https://user-images.githubusercontent.com/64135078/206057560-7e7564d8-6556-4ecb-872d-ae2a17756601.png)

6. In Jenkins UI configure Artifactory.  Configure the server ID, URL and Credentials, run Test Connection.
![p14-16](https://user-images.githubusercontent.com/64135078/206057851-f377d0ce-2c50-4b81-8385-7deb933a51d3.png)

## Phase 2 - Integrate Artifactory repository with Jenkins
1. Create a dummy `Jenkinsfile` in the `php-todo` repo. Paste the codes below into the file
   ```
   pipeline {
    agent any

    stages {

      stage("Initial cleanup") {
            steps {
              dir("${WORKSPACE}") {
                deleteDir()
              }
            }
          }

      stage('Checkout SCM') {
        steps {
              git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
        }
      }

      stage('Prepare Dependencies') {
        steps {
              sh 'mv .env.sample .env'
              sh 'composer install'
              sh 'php artisan migrate'
              sh 'php artisan db:seed'
              sh 'php artisan key:generate'
        }
      }
    }
   }
   ```
   Notice the Prepare Dependencies section

  The required file by PHP is .env so we are renaming .env.sample to .env
  Composer is used by PHP to install all the dependent libraries used by the application
  php artisan uses the .env file to setup the required database objects – (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)

2. On the database server, create database and user. This can be done in the ansible mysql role.
   ```
    Create database homestead;
    CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
    GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
   ```
   Note: the IP address of the new user should be the private IP address of the Jenkins server. And Install mysql-client on the Jenkins's server
   ```
   sudo yum install mysql -y
   ```

   Also in the database server, set the `bind-address` to `0.0.0.0`
   ```
   sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
   sudo systemctl restart mysql
   ```

3. In the `php-todo` repo, go to the `.env.sample` file, and update the following variables
   ```
   DB_HOST=<Priv-IP-Addr-DB-SERVER>
   DB_CONNECTION=mysql
   DB_PORT=3306
   ```

4. Using Blue Ocean, create a multibranch Jenkins pipeline for the `php-todo`

![p14-17](https://user-images.githubusercontent.com/64135078/206058002-09ac691e-453d-4e0e-9b99-ad7ac98cb451.png)

5. Update the `Jenkinsfile` to include Unit tests step
   ```
   stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 
   ```
![p14-18](https://user-images.githubusercontent.com/64135078/206058254-67b4e040-6033-4e6d-a301-db1c1663e17e.png)

## Phase 3 - Code Quality Analysis
This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. As a DevOps engineer, you also have a role to play. Especially when it comes to setting up the tools.

For PHP the most commonly tool used for code quality analysis is [phploc](https://phpqa.io/projects/phploc.html). [Read the article here for more](https://matthiasnoback.nl/2019/09/using-phploc-for-quick-code-quality-estimation-part-1/)

The data produced by phploc can be ploted onto graphs in Jenkins.

Install phploc, phpunit in jenkins server
```
sudo dnf --enablerepo=remi install php-phpunit-phploc
wget -O phpunit https://phar.phpunit.de/phpunit-7.phar
chmod +x phpunit
sudo yum install php-xdebug
```

1. Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.
   ```
   stage('Code Analysis') {
    steps {
          sh 'phploc app/ --log-csv build/logs/phploc.csv'

    }
   }
   ```

2. Plot the data using `plot` Jenkins plugin.
   This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots’ data series latest values are pulled from the CSV file generated by `phploc`.
```
stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }
```
![p14-19](https://user-images.githubusercontent.com/64135078/206058623-40780fa8-817a-42ec-8f42-89ea9422eccb.png)

You should now see a Plot menu item on the left menu. Click on it to see the charts. (The analytics may not mean much to you as it is meant to be read by developers. 
So, you need not worry much about it – this is just to give you an idea of the real-world implementation).

![p14-20](https://user-images.githubusercontent.com/64135078/206058791-726b9519-9101-4ec7-a0bd-a6f799ca5d04.png)

3. Bundle the application code for into an artifact (archived package) upload to Artifactory
```
stage ('Package Artifact') {
  steps {
    sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
  }
}
```

4. Publish the resulted artifact into Artifactory
```
stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }
```

![p14-21](https://user-images.githubusercontent.com/64135078/206059036-a8008c5d-f3e8-4791-b1cb-92af1de0cf2c.png)

5. Deploy the application to the dev environment by launching Ansible pipeline
```
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
```

The build job used in this step tells Jenkins to start another job. In this case it is the ansible-config-mgt job, and we are targeting the main branch. Hence, we have ansible-config-mgt/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. 
The name of the parameter is env and its value is dev. Meaning, deploy to the Development environment.

![p14-22](https://user-images.githubusercontent.com/64135078/206059592-055f03ea-6c3b-4fd8-a01f-633d0efd13cd.png)

ut how are we certain that the code being deployed has the quality that meets corporate and customer requirements? Even though we have implemented Unit Tests and Code Coverage Analysis with phpunit and phploc, we still need to implement Quality Gate to ensure that ONLY code with the required code coverage, and other quality standards make it through to the environments.

To achieve this, we need to configure SonarQube – An open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

# STEP 3 - SONARQUBE INSTALLATION


Before we start getting hands on with SonarQube configuration, it is incredibly important to understand a few concepts:

- Software Quality – The degree to which a software component, system or process meets specified requirements based on user needs and expectations.
- Software Quality Gates – Quality gates are basically acceptance criteria which are usually presented as a set of predefined quality criteria that a software development project must meet in order to proceed from one stage of its lifecycle to the next one.

SonarQube is a tool that can be used to create quality gates for software projects, and the ultimate goal is to be able to ship only quality software code.

Despite that DevOps CI/CD pipeline helps with fast software delivery, it is of the same importance to ensure the quality of such delivery. Hence, we will need SonarQube to set up Quality gates. In this project we will use predefined Quality Gates (also known as The Sonar Way). Software testers and developers would normally work with project leads and architects to create custom quality gates.

## Install SonarQube on Ubuntu 20.04 With PostgreSQL as Backend Database
I download a sonarqube role that installs sonarqube and postgresql
![p23](https://user-images.githubusercontent.com/64135078/206059740-585e2cd9-4c3f-434a-a8e3-1205f78e77f8.png)

When the pipeline is complete, access sonarqube from the browser using the <sonarqube_server_url>:9000/sonar
![p14-24](https://user-images.githubusercontent.com/64135078/206060014-b08f8e69-8bb1-4b35-b8df-f92699e9fa6c.png)

username-`admin`
password-`admin`

![p14-25](https://user-images.githubusercontent.com/64135078/206060174-c0754125-ecf7-462c-b9b3-4dfcb7ee6083.png)

# STEP 4 - CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE
- In Jenkins, install `SonarScanner` plugin

  ![](images/6-a.png)

- Navigate to configure system in Jenkins. Add SonarQube server as shown below:

  `Manage Jenkins > Configure System`

  ![](images/6-b.png)

- Generate authentication token in SonarQube

 `User > My Account > Security > Generate Tokens`

 ![](images/6-c.png)

- Configure Quality Gate Jenkins Webhook in SonarQube – The URL should point to your Jenkins server 
  `http://{JENKINS_HOST}/sonarqube-webhook/`
  `Administration > Configuration > Webhooks > Create`

  ![](images/6-d.png)

  ![](images/6-e.png)

- Setup SonarQube scanner from Jenkins – Global Tool Configuration
  ```Manage Jenkins > Global Tool Configuration```

  ![](images/7-a.png)

Update Jenkins Pipeline to include SonarQube scanning and Quality Gate
Below is the snippet for a Quality Gate stage in Jenkinsfile before the "Package Artifact Stage".
```
    stage('SonarQube Quality Gate') {
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }

        }
    }
```

NOTE: The above step will fail because we have not updated `sonar-scanner.properties

- Configure `sonar-scanner.properties` – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the `tools` directory on the server to configure the `properties` file in which SonarQube will require to function during pipeline execution.

`cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/`

Open `sonar-scanner.properties` file
`sudo vi sonar-scanner.properties`

Add configuration related to `php-todo` project
```
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
```

End-to-End Pipeline Overview

Indeed, this has been one of the longest projects from Project 1, and if everything has worked out for you so far, you should have a view like below:

![](images/7-b.png)

But we are not completely done yet!

The quality gate we just included has no effect. Why? Well, because if you go to the SonarQube UI, you will realise that we just pushed a poor-quality code onto the development environment.

- Navigate to `php-todo` project in SonarQube

  ![](images/7-c.png)

  There are bugs, and there is 0.0% code coverage. (code coverage is a percentage of unit tests added by developers to test functions and objects in the code)

- If you click on php-todo project for further analysis, you will see that there is 6 hours’ worth of technical debt, code smells and security issues in the code.
  
  ![](images/7-d.png)

In the development environment, this is acceptable as developers will need to keep iterating over their code towards perfection. But as a DevOps engineer working on the pipeline, we must ensure that the quality gate step causes the pipeline to fail if the conditions for quality are not met.

### Conditionally deploy to higher environments
In the real world, developers will work on feature branch in a repository (e.g., GitHub or GitLab). There are other branches that will be used differently to control how software releases are done. You will see such branches as:

- Develop
- Master or Main
(The * is a place holder for a version number, Jira Ticket name or some description. It can be something like Release-1.0.0)
- Feature/*
- Release/*
- Hotfix/*
etc.

There is a very wide discussion around release strategy, and git branching strategies which in recent years are considered under what is known as GitFlow (Have a read and keep as a bookmark – it is a possible candidate for an interview discussion, so take it seriously!)

Assuming a basic gitflow implementation restricts only the develop branch to deploy code to Integration environment like sit.

### Let us update our Jenkinsfile to implement this:
- First, we will include a When condition to run Quality Gate whenever the running branch is either develop, hotfix, release, main, or master
```
when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
```

- Then we add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.
```
timeout(time: 1, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
    }
```

The complete stage will now look like this:

```
 stage('SonarQube Quality Gate') {
      when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
            }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
```

To test, create different branches and push to GitHub. You will realise that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

If everything goes well, you should be able to see something like this:

![](images/8-a.png)

![](images/8-b.png)

Notice that with the current state of the code, it cannot be deployed to Integration environments due to its quality. In the real world, DevOps engineers will push this back to developers to work on the code further, based on SonarQube quality report. Once everything is good with code quality, the pipeline will pass and proceed with sipping the codes further to a higher environment.



