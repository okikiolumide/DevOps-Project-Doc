# Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

## Simulating a typical CI/CD Pipeline for a PHP Based application

This project is a continuation of infrastructure development using Ansible and will involve simulating continuous integration and delivery. Therefore the build process will be automated and the builds can be viewed by everyone, testing the quality of the code before deployment uand automate the deployment process in a clone production environment

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
