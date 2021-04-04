# Ansible Configuration Management

In this project, Ansible will be used to automate a lot of manual operations done in previous projects like to set up virtual servers, install and configure required software, deploy your web application.
It will also make me become confident at writing code using declarative language such as YAML since most of the routine tasks will be automated with Ansible Configuration Management


## Ansible Client as a Jump Server (Bastion Host)
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. In other words, it is a system on a network used to access and manage devices located in a different security zone
Ideally, the webservers would be located inside a secured network which cannot be accessed directly from the Internet. That means, even DevOps engineers cannot remotely access the servers over the internet by SSHing into the Web servers directly but can only access it through a Jump Server  providing an extra layer of security

![bastion](https://user-images.githubusercontent.com/30922643/113473839-0404dc80-9464-11eb-9e44-0446970acc0c.png)


## Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

### Step 1: Install and configure Ansible on EC2 Instance

- Update the Jenkins EC2 Instance Name tag to `Jenkins-Ansible`. This server would be used to run playbooks.
- Ccreate a new repository in Github and name it `ansible-config-mgt`.
- Install Ansible

      sudo apt update

      sudo apt install ansible
 ![ansible-install](https://user-images.githubusercontent.com/30922643/113473818-eafc2b80-9463-11eb-8cfc-f9f7d101f78a.JPG)
![ansible-install JPG-2](https://user-images.githubusercontent.com/30922643/113473823-efc0df80-9463-11eb-99b7-c0f3ed321480.JPG)
     
- Check the Ansible version 
  
      ansible --version

### Step 2: Setup Jenkinsto trigger ansible build

- Create a new Freestyle project ansible in Jenkins and point it to ‘ansible-config-mgt’ repository.
- Configure Webhook in GitHub and set webhook to trigger ansible build.
- Configure a Post-build job to save all (**) files
- Test the setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

      ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
      
      ![archive](https://user-images.githubusercontent.com/30922643/113473852-1252f880-9464-11eb-9768-98584e530865.JPG)


*Note: Trigger Jenkins project execution only for /main (master) branch.*
*To prevent reconfiguring the Github webhook with a new IP address after stop/starting the Jenkins-Ansible server, allocate an elastic IP address to the Jenkins-Ansible server

![jenkins_ansible](https://user-images.githubusercontent.com/30922643/113473598-55ac6780-9462-11eb-9b9c-45fc0036b887.png)


### Step 3: Begin Ansible Development

- Create a new branch that will be used for development of a new feature in the ansible-config-mgt GitHub repository.
*My new branch is feature/config-mgt


- Checkout the newly created feature branch to your local machine and start building the code and directory structure

![branch-create](https://user-images.githubusercontent.com/30922643/113473868-2991e600-9464-11eb-9825-9f62069e9cb0.JPG)

- Create a directory and name it `playbooks` - it will be used to store all the playbook files.
- Create a directory and name it `inventory` - it will be used to keep the hosts organised.
- Within the playbooks folder, create the first playbook, and name it `common.yml`
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) `dev`, `staging`, `uat`, and `prod` respectively.

![folders-create](https://user-images.githubusercontent.com/30922643/113473904-59d98480-9464-11eb-8c97-c38d899f016c.JPG)

![inventory-create](https://user-images.githubusercontent.com/30922643/113473893-46c6b480-9464-11eb-8f5f-f4d13b77a12b.JPG)


### Step 4: Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
The intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize the hosts in such an Inventory

*Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this you need to copy your private (.pem) key and provide a path to it so Ansible could use it to connect. Do not forget to change permissions to your private key chmod 400 key.pem, otherwise EC2 will not accept the key. Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.*

Update your inventory/dev.yml file with this:

      [nfs]
      <NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

      [webservers]
      <Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
      <Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

      [db]
      <Database-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

      [lb]
      <Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu' ansible_ssh_private_key_file=<path-to-.pem-private-key>
      
      ![dev](https://user-images.githubusercontent.com/30922643/113473882-39a9c580-9464-11eb-8716-c48fd71d927f.JPG)

- Test all hosts are reachable using
      
      ansible -m ping all
 
 ![ping-all-test](https://user-images.githubusercontent.com/30922643/113473926-82617e80-9464-11eb-9ab2-e25967da8d00.JPG)

 
### Step 5: Create a Common Playbook

In `common.yml` playbook write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update the `playbooks/common.yml` file with following code:

                  ---
                  - name: update web, nfs and db servers
                    hosts: webservers, nfs, db
                    remote_user: ec2-user
                    become: yes
                    become_user: root
                    tasks:
                    - name: ensure wireshark is at the latest version
                      yum:
                        name: wireshark
                        state: latest

                  - name: update LB server
                    hosts: lb
                    remote_user: ubuntu
                    become: yes
                    become_user: root
                    tasks:
                    - name: ensure wireshark is at the latest version
                      apt:
                        name: wireshark
                        state: latest
                        
  Feel free to update this playbook with following tasks:

Create a directory and a file inside it
Change timezone on all servers
Run some shell script               

### Step 6: Update GIT with the latest code

Commit your code into GitHub:

use git commands to add, commit and push your branch to GitHub.

            git status

            git add <selected files>

            git commit -m "commit message"
            
- Create a Pull request (PR)
- Wear a hat of another developer for a second, and act as a reviewer.
- If the reviewer is happy with your new feature development, merge the code to the master branch.
- Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
- Once your code changes appear in master branch - Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server.

### Step 7: Run first Ansible test

 Execute `ansible-playbook` command and verify if your playbook actually works using:
 
       sudo ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml

Go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`

![wireshark -v](https://user-images.githubusercontent.com/30922643/113473985-e1bf8e80-9464-11eb-92b7-60a11dc8c8b6.JPG)


The updated Ansible architecture now looks like this:

![image](https://user-images.githubusercontent.com/30922643/113473729-424dcc00-9463-11eb-9b9d-3361c729a010.png)

*Optional step - Repeat once again, 
Update your ansible playbook with some new Ansible tasks and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!*

**Creating a test file in /var/www on webservers**

![test-file](https://user-images.githubusercontent.com/30922643/113474000-fa2fa900-9464-11eb-9340-74bd9e613639.JPG)

## PROJECT DEMONSTRATION
https://drive.google.com/drive/folders/1qCtI-joKXGmr2p5DRPX13-ta3WoXyGQO?usp=sharing

## GITHUB REPO LINK
https://github.com/okikiolumide/ansible-config-mgt

## BLOCKERS
1. Error when running ansible playbooks - `Fatal: [172.31.3.218]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ec2-user@172.31.3.218: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}`
- Solution: Remove "sudo" when running ansible playbook command i.e  ~~sudo~~ `ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml`
      
 2. Error when trying to ping host - `Failed to connect to the host via ssh: Host key verification failed`.
- Solution: Added `"host_key_checking = False"` to ansible.cfg file

3. Error in Jenkins when building - Couldn't find any revision to build. Verify the repository and branch configuration for this job
- Solution: Edited Project configuration settings in Jenkins by changing the repository name from `/master` to `/main`

4. Error when trying to copy public key to servers - `Permission denied (public key)`
- Solution: I had to manually copy the keys to /.ssh/authorised_keys on each server



## RESOURCES
1. https://darey.io/
2. https://docs.ansible.com/
3. https://www.reddit.com/r/ansible/comments/g5djak/need_help_with_running_playbook/
4. https://askubuntu.com/questions/311558/ssh-permission-denied-publickey
5. https://www.digitalocean.com/
6. https://phoenixnap.com/



