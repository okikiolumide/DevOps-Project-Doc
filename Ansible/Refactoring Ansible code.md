# Ansible Refactoring & Static Assignments (Imports)

In this project some improvements would be made to the code in the ansible-config-mgt repository. This will involve refactoring of Ansible code, creating assignments, and learning how to use the imports functionality. 
The Imports functionality is used to effectively utilize previously created playbooks in a new playbook, this improves the organisation of tasks and reuse the playbooks when needed. The Project Architecture is

![image](https://user-images.githubusercontent.com/30922643/115089168-9b107080-9f09-11eb-990a-4971ab7dfd20.png)


## TASKS
1. Enhancement of Project with Jenkins CI
2. Refactor Ansible code by importing other playbooks
3. Configure UAT Webservers with a role ‘Webserver’
4. Reference ‘Webserver’ role
5. Commit & Test

## Project Implementation - Code Refactoring

Refactoring is a general term in computer programming. It means making changes to the source code without changins expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

### Step 1 - Jenkins job enhancement

- Go to the Jenkins-Ansible server and create a new directory called ansible-config-mgt for storing all artifacts after each build.

      mkdir /home/ubuntu/ansible-config-mgt
      
- Change permissions to this directory, so Jenkins could save files there  
      
      chmod -R 0777 /home/ubuntu/ansible-config-mgt
      
- Go to Jenkins web console -> `Manage Jenkins` -> `Manage Plugins` -> on Available tab search for `Copy Artifact` and install this plugin without restarting Jenkins

- Create a new Freestyle project and name it `save_artifacts`.

- This project will be triggered by completion of the existing ansible project. Configure it accordingly:

![image](https://user-images.githubusercontent.com/30922643/115089315-df037580-9f09-11eb-8189-0bc97118dd84.png)

![image](https://user-images.githubusercontent.com/30922643/115089272-d14df000-9f09-11eb-95f0-27dc539b2cfe.png)

![image](https://user-images.githubusercontent.com/30922643/115089337-efb3eb80-9f09-11eb-9304-ccdd3d170db7.png)

*Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.*

- Test the set up by making some change in README.MD file inside the ansible-config-mgt repository (right inside master branch)

![image](https://user-images.githubusercontent.com/30922643/115089394-0b1ef680-9f0a-11eb-941c-8244ce1e3ef1.png)


![image](https://user-images.githubusercontent.com/30922643/115089364-fcd0da80-9f09-11eb-8c2a-1f0d76c6f414.png)

- If both Jenkins jobs have completed one after another - files would be located inside /home/ubuntu/ansible-config-mgt directory and it will be updated with every commit to the master branch.


## Step 2 - Refactor Ansible code by importing other playbooks into site.yml

- Within playbooks folder, create a new file and name it site.yml - This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.

- Create a new folder in root of the repository and name it `static-assignments`. The static-assignments folder is where all other children playbooks will be stored for easy organization. 

- Move common.yml file into the newly created static-assignments folder.

- Inside site.yml file, import common.yml playbook.

      ---
      - hosts: all
      - import_playbook: ../static-assignments/common.yml

Tree

        ├── static-assignments
        │   └── common.yml
        ├── inventory
        └── dev
        └── stage
        └── uat
        └── prod
        └── playbooks
            └── site.yml
            
   ![image](https://user-images.githubusercontent.com/30922643/115076587-ad33e400-9ef4-11eb-9ed1-48f527853e6f.png)

   
- Run ansible-playbook command against the dev environment

- Create another playbook under static-assignments and name it common-del.yml to configure deletion of wireshark utility in the servers.

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


- update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:

      ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml

- Make sure that wireshark is deleted on all the servers by running wireshark --version
![image](https://user-images.githubusercontent.com/30922643/115090628-40791380-9f0d-11eb-8cc1-9366ddc6a4a0.png)


## Step 3 - Configure UAT Webservers with a role ‘Webserver’

- Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly - Web1-UAT and Web2-UAT.
- To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.
- Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)

            mkdir roles
            cd roles
            ansible-galaxy init webserver
            
            
![image](https://user-images.githubusercontent.com/30922643/115087194-a497d980-9f05-11eb-976c-d056a03cd47f.png)

![image](https://user-images.githubusercontent.com/30922643/115087249-ba0d0380-9f05-11eb-9d33-9acc41b39116.png)


- After removing unnecessary directories and files, the roles structure should look like this

            └── webserver
                ├── README.md
                ├── defaults
                │   └── main.yml
                ├── handlers
                │   └── main.yml
                ├── meta
                │   └── main.yml
                ├── tasks
                │   └── main.yml
                └── templates

- Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers

            [uat-webservers]
            <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
            <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

- Test the connection to uat-webservers by using the ping command

![image](https://user-images.githubusercontent.com/30922643/115087332-de68e000-9f05-11eb-8b1a-fec574fc26ec.png)

- In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

- Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

     >Install and configure Apache (httpd service)
            
     >Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
      
     > Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
     
     > Make sure httpd service is started
      
    The `main.yml` 
    
   ![image](https://user-images.githubusercontent.com/30922643/115088207-9480f980-9f07-11eb-8c05-67f9128da1b4.png)
   
      
## Step 4 - Reference ‘Webserver’ role

 create a new assignment for uat-webservers uat-webservers.yml within the static-assignments folder to reference the role.

            ---
            - hosts: uat-webservers
              roles:
                 - webserver
     
*Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml*

So, we should have this in site.yml

            ---
            - hosts: all
            - import_playbook: ../static-assignments/common.yml

            - hosts: uat-webservers
            - import_playbook: ../static-assignments/uat-webservers.yml



## Step 5 - Commit & Test

Commit the changes, create a Pull Request and merge them to main branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory

            ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml
        
  ![image](https://user-images.githubusercontent.com/30922643/115086170-aa8cbb00-9f03-11eb-9d6d-cec9265db903.png)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

            http://<Web1-UAT-Server-Private-IP-or-Private-DNS-Name>/index.php

or

            http://<Web1-UAT-Server-Private-IP-or-Private-DNS-Name>/index.php


![image](https://user-images.githubusercontent.com/30922643/115086327-ee7fc000-9f03-11eb-9729-b57b29ba8da3.png)


## GITHUB REPO
https://github.com/okikiolumide/ansible-config-mgt

## PROJECT DEMONSTRATION
https://drive.google.com/drive/folders/1WmrGZhdQPoQBc_FUVaY7nDcQAJh_pwSS?usp=sharing

## BLOCKERS
No Blocker encountered

## RESOURCES
1. https://darey.io
2. https://docs.ansible.com/
3. https://docs.github.com/en


