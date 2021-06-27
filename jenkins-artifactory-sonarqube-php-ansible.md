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
3. Create a new pipeline
4. Select GitHub
5. Connect Jenkins with GitHub
6. Login to GitHub & Generate an Access Token
7. Copy Access Token
8. Paste the token and connect
9. Create a new Pipeline

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
            ansiblePlaybook(credentialsId: 'awsprivatekey', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml', tags: '${tag}')
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

![ansible-playbook](https://user-images.githubusercontent.com/30922643/123554021-139b7500-d776-11eb-8001-a6a5090bbef8.png)

