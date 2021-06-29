
# Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

## Simulating a typical CI/CD Pipeline for a PHP Based application

This project is a continuation of infrastructure development using Ansible and will involve simulating continuous integration and delivery. Therefore the build process will be automated and the builds can be viewed by everyone, the quality of the code will be tested before deployment and deployment process will be automated into a clone production environment

![image](https://user-images.githubusercontent.com/30922643/123549349-8f8ac280-d760-11eb-8436-21503760f4d8.png)

*Note: Two more roles will be added to the already existing Ansible repo:*
  - **Sonarqube**: SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used
to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities
  - **Artifactory**: Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural
extension to the source code repository, in that the outcome of your build process is stored. It can be used for certain
other automation, but we will it strictly to manage our build artifacts.

## Tasks
  1.  Setting up inventory
  2.  Configuring Ansible For Jenkins Deployment
  3.  Configuring Artifactory
  4.  Configuring Sonarqube

## Ansible Inventory

      ci
      dev
      pentest
      pre-prod
      prod
      sit
      uat
      
**ci inventory file**

      [jenkins]
      <Jenkins-Private-IP-Address>
      
      [nginx]
      <Nginx-Private-IP-Address>
      
      [sonarqube]
      <SonarQube-Private-IP-Address>
      
      [artifact_repository]
      <Artifact_repository-Private-IP-Address>
      
**dev Inventory file**
  
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
      
**pentest inventory file**

      [pentest:children]
      pentest-todo
      pentest-tooling
      
      [pentest-todo]
      <Pentest-for-Todo-Private-IP-Address>
      
      [pentest-tooling]
      <Pentest-for-Tooling-Private-IP-Address>     


## Configuration of Ansible for Jenkins Deployment
For running ansible playbooks from Jenkins UI instead of manually running ansible playbooks on CLI

1. Navigate to Jenkins URL
2. Install & Open Blue Ocean Jenkins Plugin

![blue-ocean](https://user-images.githubusercontent.com/30922643/123769291-0dfe7600-d8c1-11eb-8437-a0fde44d78a0.PNG)

3. Create a new pipeline

![setting up pipeline](https://user-images.githubusercontent.com/30922643/123769361-266e9080-d8c1-11eb-86cd-559716726bf4.PNG)

4. Select GitHub
5. Connect Jenkins with GitHub
6. Login to GitHub & Generate an Access Token
7. Copy Access Token
8. Paste the token and connect
9. Create a new Pipeline 

![blue-ocean branch](https://user-images.githubusercontent.com/30922643/123769522-4b630380-d8c1-11eb-8392-1f949e841c8c.PNG)


*At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some
guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to
exit the Blue Ocean console.*

### - Creating JenkinsFile
- Create a deploy folder in the Ansible-project, and crerate a file named Jenkinsfile within the deploy folder

![tree](https://user-images.githubusercontent.com/30922643/123553057-25c6e480-d771-11eb-9720-869d177cb9a5.PNG)

- Add the following snippet

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

- Go into the Ansible pipeline in Jenkins, and select configure
- Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/
Jenkinsfile
- Back to the pipeline again, this time click “Build now”
- To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from
Blue Ocean interface.
- You could trigger the build again by clicking the play button against the branch, then click the branch.

Since our pipeline is multibranch, we could build all the branches in the repo independently. To see this in action,

   - Create a new git branch and name it features/jenkinspipeline-stages
   - Add a new build stage "Test"

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
    
- To make the new branch show in Jenkins UI, click Administration to exit Blue Ocean, click the project and click Scan Repository Now from the left pane
- Refresh the page and you should see the new branch.
- Open Blue Ocean and you should see the new branch building (or has finished building)

![added test-stage](https://user-images.githubusercontent.com/30922643/123769617-633a8780-d8c1-11eb-9ce0-e9e838119b09.PNG)

**Quick task**
  1. Create a pull request to merge the latest code into the `main branch`
  2. After merging the `PR`, go back into your terminal and switch into the `main` branch.
  3. Pull the latest change.
  4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an `echo` command like we have in `build` and `test` stages)
     1. Package 
     2. Deploy 
     3. Clean up
  5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
  6. Eventually, your main branch should have a successful pipeline like this in blue ocean

![more-stage build](https://user-images.githubusercontent.com/30922643/123553407-e5686600-d772-11eb-9d9b-1df24259b8d0.PNG)

### Running Ansible playbook from Jenkins
- Install Ansible on your Jenkins server

      sudo apt install ansible -y
- Install Ansible plugin in Jenkins UI
    - Go to Manage Jenkins
    - Click Manage Plugins
    - Click Available tab and in the search bar, enter Ansible and click the check box next to the plugin
    - Scroll down and click 'Install without restart'
    - Create Jenkinsfile from scratch (delete all the current stages in the file)
    
- Ensure that Ansible runs against the Dev environment successfully. Add a new stage to clone the GitHub repo

        stage('Clone repo') {
          steps {
            git(branch: 'feature/jenkinspipeline', url: 'https://github.com/okikiolumide/ansible-config-mgt.git')
          }
        }
This step will clone the repo from the feature/jenkinspipeline branch.

*Remember that ansible.cfg must be exported to environment variable so that Ansible knows where to find
Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles
will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update
the section roles_path each time there is an execution. You may not have this issue if you run only from the
main branch.*

- Add another stage to edit /etc/ansible/ansible.cfg file
 
        stage('Update ansible.cfg') {
        steps {
          sh ('sudo sed -i -e "/roles_path = \\/[a-z]*\\/*/a\\roles_path = $cpath" -e "/roles_path = \\/[a-z]*\\/*/d" /etc/ansible/ansible.cfg')
        }
      }

*Possible errors to watch out for:*
*1. Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub
has discontinued the use of Master due to Black Lives Matter. You can read more here)*
*2. Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file
alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in
there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create
environment variables to set*

- Add jenkins user to the sudoers file with permissions to use the sudo command without password when running playbooks.
    
    Run `sudo visudo` and add the following line
        
        jenkins ALL=(ALL) NOPASSWD: ALL

- Run the playbook by adding this stage

        stage('Run playbook') {
          steps {
            ansiblePlaybook(credentialsId: 'awsprivatekey', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/dev', playbook: 'playbooks/site.yml', tags: '${tag}')
          }
        }
*Note: Remember to Add the Jenkins-Ansible instance private key to Jenkins credentials* 

- Add a stage to cleanup the workspace after every build whether the build failed, was successful or aborted etc

        stage('Clean up') {
          steps {
            cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
          }
        }
        
       
- Commit and push the code to repo. This should trigger a build automatically on Jenkins if the Github webhook has been properly configure

  ![ansible-playbook](https://user-images.githubusercontent.com/30922643/123768686-739e3280-d8c0-11eb-94f8-7c444928da3a.png)
![ansible-playbook-pipeline](https://user-images.githubusercontent.com/30922643/123768585-5e290880-d8c0-11eb-847c-8500988a3ce9.png)



## Parameterizing Jenkinsfile for Ansible Development

To deploy to other environments like sit,uat,pentest etc, we need to make use of parameters

- Update SIT environment inventory file with new servers

        [tooling]
        <SIT-Tooling-Web-Server-Private-IP-Address>

        [todo]
        <SIT-Todo-Web-Server-Private-IP-Address>

        [nginx]
        <SIT-Nginx-Private-IP-Address>

        [db:vars]
        ansible_user=ec2-user
        ansible_python_interpreter=/usr/bin/python

        [db]
        <SIT-DB-Server-Private-IP-Address>

- Update Jenkinsfile to introduce parameterization. Below is just one parameter. It has a default value in
case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.

          pipeline {
              agent any
              parameters {
                  string(name: 'inventory', defaultValue: 'dev', description: 'This is the
          ˓→inventory file for the environment to deploy configuration')
          }
        }
Parameters in the stage can be used to change the inventory environment by changing the hard coded environment from `inventory/dev` to `'inventory/${inventory}'`. From now on, each time the build is executed, it will expect an input. Once the desired environment is entered, hit Run and the configuration will be deployed to the specific environment.

![parameter](https://user-images.githubusercontent.com/30922643/123769154-e60f1280-d8c0-11eb-88ae-26268aec8341.PNG)


## Setting up CI/CD Pipeline for a php TODO Application
A PHP application is introduced to add to the list of software products been managed in the  infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.
The goal  is to deploy the application onto servers directly from Artifactory rather than from git

### Configuration of Artifactory with Jenkins
- Create an Ansible role to install Artifactory (ignore the Nginx part)
- Run the Ansible role against the Artifactory server

![artifactory-install-az](https://user-images.githubusercontent.com/30922643/123770957-96c9e180-d8c2-11eb-8c83-b4ef4ffba937.PNG)


**Phase 1 Prepare Jenkins**
- Fork the repository below into your GitHub account
https://github.com/darey-devops/php-todo.git
- On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update Ansible accordingly later)

        sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}

- Install the following Jenkins plugins
  - Plot Plugins: This will be used to display tests reports, and code coverage information.
  - Artifactory plugins: This will be used to easily upload code artifacts into an Artifactory server.

- Configure Artifactory in Jenkins UI
  - Click Manage Jenkins, click Configure System
  - Scroll down to JFrog, click Add Artifactory Server
  - Enter the Server ID
  - Enter the URL as: http://<artifactory-server-ip>:8082/artifactory

  ![jfrog-landing](https://user-images.githubusercontent.com/30922643/123770866-89145c00-d8c2-11eb-8191-eaa479779804.PNG)

  
**Phase 2 - Integrate Artifactory repository with Jenkins**
- Create a dummy `Jenkinsfile` in the repository
- Using Blue Ocean, create a multibranch Jenkins pipeline
- On the database server, create database and user

      Create database homestead;
      CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
      GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';

  
  ![show homestead tables](https://user-images.githubusercontent.com/30922643/123771792-41da9b00-d8c3-11eb-9422-ab683d0b1d1d.PNG)

- Update the db connection details in the .env.sample file within the todo app 
- Update `Jenkinsfile` with proper pipeline configuration

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

*Note: Notice the **Prepare Dependencies** section*
• *The required file by PHP is .env so we are renaming .env.sample to .env*
• *Composer is used by PHP to install all the dependent libraries used by the application*
• *php artisan uses the .env file to setup the required database objects - (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)*

  ![prepare depencies](https://user-images.githubusercontent.com/30922643/123770196-fecbf800-d8c1-11eb-809c-c97538574cd8.PNG)
  
- Add unit tests to `Jenkinsfile` 

      stage('Execute Unit Tests') {
          steps {
                sh './vendor/bin/phpunit'
          }
  
![execute unit tests](https://user-images.githubusercontent.com/30922643/123771186-c37df900-d8c2-11eb-8478-aa307dea5c7f.PNG)


  
**Phase 3 - Code Quality Analysis**
For PHP, the most commonly tool used for code quality analysis is phploc. As a DevOps engineer, we assist when it comes to setting up the tools. The data produced by phploc can be ploted onto graphs in Jenkins. 
*Note: Unit Tests and Code Coverage Analysis is implemented with phpunit and phploc*

- Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/
phploc.csv file.

        stage('Code Analysis') {
              steps {
              sh 'phploc app/ --log-csv build/logs/phploc.csv'
              }
           }

-  Plot the data using plot Jenkins plugin.

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

- Click on the Plot menu item on the left menu to see the charts.

- Upload the artifact (archived package) into artifactory

            stage ('Package Artifact') {
            steps {
            sh 'zip -qr ${WORKSPACE}/php-todo.zip ${WORKSPACE}/*'
            }

- Publish packaged artifact into Artifactory

          stage ('Deploy Artifact') {
            steps {
                    script { 
                        def server = Artifactory.server 'artifactory-server'
                        def uploadSpec = """{
                            "files": [{
                              "pattern": "php-todo.zip",
                              "target": "php-todo"
                            }]
                        }"""

                        server.upload(uploadSpec) 
                      }
            }
          }

-  Deploy the application to the dev environment by launching Ansible pipeline

          stage ('Deploy to Dev Environment') {
            steps {
            build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
            }
          }


The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch. Hence, we have ansible-project/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is `env` and its value is `dev`. This means deploy to the Development environment

  ![phploc plots](https://user-images.githubusercontent.com/30922643/123768946-afd19300-d8c0-11eb-9e56-b3ac0699f511.PNG)
![plot code coverage report](https://user-images.githubusercontent.com/30922643/123768954-b233ed00-d8c0-11eb-8898-b1b3f0d0d091.PNG)


### Step 3: Configure SonarQube

Sonarqube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells,
and security vulnerabilities. THe goal is to ship quality code

- **Software Quality** - The degree to which a software component, system or process meets specified requirements
based on user needs and expectations.
- **Software Quality Gates** - Quality gates are basically acceptance criteria which are usually presented as a set of predefined quality criteria that a software development project must meet in order to proceed from one stage of its lifecycle to the next one.
We will be using a predefined Quality Gates (also known as The Sonar Way). 

**Install SonarQube on Ubuntu 20.04 With PostgreSQL as Backend Database**
Here is a manual approach to installation.
*Side-Tasks: Automate with Ansible*

- Tune Linux Kernel

      sudo sysctl -w vm.max_map_count=262144
      sudo sysctl -w fs.file-max=65536
      ulimit -n 65536
      ulimit -u 4096

  ![linux kernel](https://user-images.githubusercontent.com/30922643/123771284-d1337e80-d8c2-11eb-8819-7be1bd7d28eb.PNG)

 - To make a permanent change, edit the file /etc/security/limits.conf and append the below

      sonarqube   -   nofile   65536
      sonarqube   -   nproc    4096
  
![update etc-security](https://user-images.githubusercontent.com/30922643/123771338-db557d00-d8c2-11eb-8f86-ac18135b68d2.PNG)

  
- Update and upgrade system packages

      sudo apt-get update
      sudo apt-get upgrade

- Install wget and unzip packages

      sudo apt-get install wget unzip -y

- Install OpenJDK and Java Runtime Environment (JRE)

      sudo apt-get install openjdk-11-jdk -y
      sudo apt-get install openjdk-11-jre -y

- To set default JDK or switch to OpenJDK, run the command,
      
      sudo update-alternatives --config java

- Verify JAVA is installed by running:
      
      java -version

  ![java version](https://user-images.githubusercontent.com/30922643/123771890-5454d480-d8c3-11eb-8cf5-eaacde89be2c.PNG)

  
**Install and Setup PostgreSQL 10 Database for SonarQube**

- Add PostgreSQL to repo list

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

- Download PostgreSQL software

      wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

- Install PostgreSQL

      sudo apt-get -y install postgresql postgresql-contrib

- Start and enable PostgreSQL Database service
  
      sudo systemctl start postgresql
      sudo systemctl enable postgresql

- Change the default password for postgres user
  
      sudo passwd postgres

- Switch to postgres user

      su - postgres

- Create a new user for SonarQube

      createuser sonar
    
- Switch to PostgreSQL shell
        
      psql

- Set up encrypted password for newly created user

      ALTER USER sonar WITH ENCRYPTED password 'sonar';

- Create a DB for SonarQube
    
      CREATE DATABASE sonarqube OWNER sonar;
- Grant required privileges
      
      grant all privileges on DATABASE sonarqube to sonar;
      Exit the shell
      \q

- Switch back to initial user
  
      exit

**Install and configure SonarQube on Ubuntu 20.04 LTS**
- Download the installation files into `tmp` directory

      cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip

  ![installation files](https://user-images.githubusercontent.com/30922643/123771979-6a629500-d8c3-11eb-8086-aaadb7219292.PNG)

  
- Unzip the archive to the `/opt directory` and move the extracted setup to `/opt/sonarqube` directory

      sudo unzip sonarqube-7.9.3.zip -d /opt
      sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube

Since Sonarqube cannot run as a root user because it will stop automatically, Create a separate group and a user to run Sonarqube

- Create a group sonar

      sudo groupadd sonar

- Add a sonar user with control over /opt/sonarqube

      sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar 
      sudo chown sonar:sonar /opt/sonarqube -R

- Open and edit SonarQube configuration file.

      sudo vim /opt/sonarqube/conf/sonar.properties
      
- Find and edit the following lines, provide the values of PostgreSQL Database username and password

      sonar.jdbc.username=sonar
      sonar.jdbc.password=sonar
      sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

- Edit sonar script and set RUN_AS_USER=sonar

      sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh

Add sonar user to sudoers file
First set the user's password to something you can easily remember
sudo passwd sonar

- Run the following command and add the following line

      sudo visudo
      sonar ALL=(ALL) NOPASSWD: ALL

- Switch to the sonar user, switch to script directory and start the script

      su - sonar
      cd /opt/sonarqube/bin/linux-x86-64/
      ./sonar.sh start


- Check SonarQube logs

      tail /opt/sonarqube/logs/sonar.log

  ![sonarqube log](https://user-images.githubusercontent.com/30922643/123771426-ed372000-d8c2-11eb-9839-792cc81fc8e6.PNG)

  
**Configure SonarQube to run as a systemd service**

Stop the running sonar service
  
        cd /opt/sonarqube/bin/linux-x86-64/

Run the script to start 
        ./sonar.sh stop

Create systemd service file

sudo vim /etc/systemd/system/sonar.service

Paste the configuration below to `systemd`:

        [Unit]
        Description=SonarQube service
        After=syslog.target network.target

        [Service]
        Type=forking

        ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
        ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

        User=sonar
        Group=sonar
        Restart=always

        LimitNOFILE=65536
        LimitNPROC=4096

        [Install]
        WantedBy=multi-user.target

  ![sonar systemstd](https://user-images.githubusercontent.com/30922643/123770116-e825a100-d8c1-11eb-961a-cf5980efde39.PNG)

  
- Save the file and Use systemd to manage the service

      sudo systemctl start sonar
      sudo systemctl enable sonar
      sudo systemctl status sonar

  ![sonar service](https://user-images.githubusercontent.com/30922643/123770062-da701b80-d8c1-11eb-9b8e-e4ebd12d6f8f.PNG)

  
**Access SonarQube**

To access SonarQube using browser, type server’s IP address followed by port 9000: `http://server_IP:9000` OR `http://localhost:9000`
username:admin password:admin

  ![sonar dashboard](https://user-images.githubusercontent.com/30922643/123770035-d0e6b380-d8c1-11eb-8d9d-f27063984931.PNG)

  
Remember to open the port in the security group for the Sonarqube instance

**Configure SonarQube and Jenkins For Quality Gate**

- Install Sonarqube in Jenkins
- Navigate to Manage Jenkins > Configure System and add a SonarQube server

    - In the name field, enter sonarqube
    - For server URL, enter the IP of your SonarQube instance
- Generate authentication tokens to use in the Jenkins UI

    - In SonarQube UI, navigate to User > My Account > Security > Generate tokens
    - Type in the name of the token and click Generate

- Configure SonarQube webhook for Jenkins
    - Navigate to Administration > Configuration > Webhooks > Create
    - Enter the name
    - Enter URL as http://<jenkins-server-ip>:8080/sonar-webhook/

- Setup SonarScanner for Jenkins
    - In Jenkins UI, go to Manage Jenkins > Global Tool Configuration
    - Look for SonarQube Scanner
    - Click 'Add SonarQube Scanner' and enter the scanner name as 'SonarQubeScanner'
    - Check the 'Install automatically' box
    - Select 'Install from Maven Central'

- Add the following build stage for Quality Gate

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

- Configure sonar-scanner.properties 
  
          cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/
          ˓→SonarQubeScanner/conf/

- Open the sonar-scanner.properties and add configuration related to php-todo project

            sonar.host.url=http://<SonarQube-Server-IP-address>:9000
            sonar.projectKey=php-todo
            #----- Default source code encoding
            sonar.sourceEncoding=UTF-8
            sonar.php.exclusions=**/vendor/**
            sonar.php.coverage.reportPaths=build/logs/phploc.csv
            sonar.php.tests.reportPath=reports/unitreport.xml

*HINT: To know what exactly to put inside the sonar-scanner.properties file, SonarQube has a configurations page where one can get some directions.*

**Update Jenkins Pipeline to include SonarQube scanning and Quality Gate**

- run `php --ini` to see loaded volumes and php config files
- search for `xdebug` using `grep xdebug` among the outputs, if it is not there install 

        sudo apt install php-xdebug

- Check the php config file and update the xdebug file:

          php --ini | grep xdebug

- Find he uncommented line `xdebug.mode = develop` and change it to: `xdebug.mode=coverage`      

- Restart php 

          sudo systemctl restart php-fpm

- Change the sonar.php.coverage.reportPaths to build/coverage-*.xml

- Run the job again

![sonar gate pipeline](https://user-images.githubusercontent.com/30922643/123769850-9f6de800-d8c1-11eb-8ae0-46d34cb2e62e.PNG)
![sonar dashboard latest](https://user-images.githubusercontent.com/30922643/123769860-a268d880-d8c1-11eb-9190-0ead26e2345d.PNG)

### Conditionally deploy to higher environments

Assuming a basic gitflow implementation restricts only the develop branch to deploy code to Integration environment like sit.
Update the Jenkinsfile to implement this:

- First, we will include a When condition to run Quality Gate whenever the running branch is either develop,
hotfix, release, main, or master

          when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}

- Add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.

          timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
          }

The complete stage should look like this:

              stage('SonarQube Quality Gate') {
                when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator:
                  ˓→"REGEXP"}
                  environment {
                  scannerHome = tool 'SonarQubeScanner'
                  }
                  steps {
                  withSonarQubeEnv('sonarqube') {
                  sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.
                  ˓→properties"
                  }
                  timeout(time: 1, unit: 'MINUTES') {
                  waitForQualityGate abortPipeline: true
                  }
                }
              }

Test this by creating different branches and push to GitHub to see that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

  ![sonar gate pipeline 2](https://user-images.githubusercontent.com/30922643/123769827-97ae4380-d8c1-11eb-8670-f5c1062f60e4.PNG)

  
### Complete the following tasks to finish the Project

1. Introduce Jenkins agents/slaves - Add 2 more servers to be used as Jenkins slave. Configure Jenkins to run its
pipeline jobs randomly on any available slave nodes.

- Spin up a new EC2 Instance

    - Install Java and Jenkins
    - Install necessary packages from Steps 1.4, 2.3 Blocker and 2.4
    - Create new user to be used by jenkins

            sudo useradd -d /var/lib/jenkins jenkins

            sudo passwd jenkins

    - Create the default root directory and set permissions

            sudo mkdir /var/lib/jenkins
            
            sudo chown -R jenkins:jenkins /var/lib/jenkins

    - Generate SSH Key for jenkins to use when logging in

            su - jenkins

            ssh-keygen -t rsa -C "Jenkins agent key"

    - Add id_rsa.pub to authorized_keys file

            # cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
            
            # chmod 600 ~/.ssh/authorized_keys

    - Copy SSH private key to clipboard

            cat ~/.ssh/id_rsa

- On the main Jenkins server

    - Navigate to Manage Jenkins > Manage Nodes

    - Click New Node
    - Enter name of the node and click the 'Permanent Agent' button and click the OK button
    - Fill in the remote root directory as /var/lib/jenkins
    - Set 'Host' value as the IP of the slave node
    - For Launch Method, select Launch Agents via SSH
    - Add new Jenkins SSH with username and private key credentials with username as jenkins and private key as the private key you copied from the node
    - For Host Key Verification Strategy, select Manually trusted key validation strategy
    - Click Save


2. Configure webhook between Jenkins and GitHub to automatically run the pipeline when there is a code push.
3. Deploy the application to all the environments
4. Optional - Experience pentesting in pentest environment by configuring Wireshark there and just explore for
information sake only. Watch Wireshark Tutorial here
• Ansible Role for Wireshark:
– https://github.com/ymajik/ansible-role-wireshark (Ubuntu)
– https://github.com/wtanaka/ansible-role-wireshark (RedHat)

## Project Code Repo
https://github.com/okikiolumide/php-todo.git  

## Resources
1. https://darey.io
2. https://docs.sonarqube.org/latest/analysis/coverage/
3. https://medium.com/linkbynet/code-quality-testing-with-sonarqube-and-gitlab-ci-for-php-applications-f0c953f4133d

4. https://www.howtoforge.com/tutorial/ubuntu-jfrog/
5. https://medium.com/linkbynet/code-quality-testing-with-sonarqube-and-gitlab-ci-for-php-applications-f0c953f4133d

6. https://xdebug.org/docs/install
