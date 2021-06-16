# DEPLOYMENT OF LAMP (LINUX APACHE MYQSQL PHP) STACK

## SOFTWARE DEVELOPMENT LIFE CYCLE

Software Development Life Cycle (SDLC) also referred to as the Application Development Life-Cycle is the standard business practices used by the software industry to design, develop and test high quality softwares. It is aimed at reducing waste and increasing the efficiency and speed of the development process. The SDLC is a measure that ensure the project is tracked and controlled




- **Stage 1: Planning phase** - This involves clearly defining the scope and purpose of the application to be developed. It directs the course and provisions the team by setting boundaries to effectively create the software, keeping the project from deviating from its original purpose. This stage gives a clearer picture of the scope of the entire project and the anticipated issues, opportunities, and directives which triggered the project.
- **Stage 2: Defining Requirements** - Defining Requirements helps to determine what the application is supposed to do and its requirements. It also determines the resources needed to build the project. 
- **Stage 3: System Design** - This describes the transformation of the software requirements into a logical structure. This contains a detailed and complete set of specifications that can be implemented in a programming language.
- **Stage 4: Building Phase** - This is when the actual development starts and the project is built. It is also known as the implementation stage where the design is transformed into a source code through coding.  
- **Stage 5: Testing Phase** - This stage refers to testing of the product where product defects are reported, tracked, fixed and retested until the product reaches the quality standards defined in the project requirement.
- **Stage 6: Deployment and Maintenance** -  Once the software testing phase is over and no bugs or errors left in the system then the final deployment process starts. As soon as customers start using the developed system after deployment, bugs are reported because of some scenarios which are not tested at all. This will result in upgrading the deployed application to a newer version and some new features can also be added into the existing software.

## SDLC MODELS

1. **Waterfall Model** - This is the first process model widely-accepted and used for software development. In this model, the whole process of the software development is divided into various phases of SDLC i.e. the outcome of one phase acts as the input of the next phase. This means that any development phase can only begin when the previous phase has been completed.The advantage of the waterfall model is processes are easily documented because the stages are clearly defined. One of the major drawbacks of this model is it is not a good model for complex and object-oriented projects because it has a high amount of risks and uncertainties.

2. **Iterative Model** - Iterative process starts with a simple implementation of a small set of the software requirements and iteratively enhances the evolving versions until the complete system is implemented and ready to be deployed.

3. **Spiral Model** - The Spiral Model combines iterative development process model and sequential linear model. It is a risk-driven model and it helps teams adopt elements of one or more process models like a waterfall, incremental, waterfall, etc.The major disadvantage is the management and process is complex

4. **V-Model** - The V-model is also known as Verification and Validation model. It is an extension of the waterfall model and is based on the association of a testing phase for each corresponding development stage. This means that the testing stage is included in every stage of the software development cycle. It works well for smaller projects where requirements are very well understood but it is a poor model for long and ongoing projects.

5. **Big Bang Model** - This model focuses on the various types of resources in the  application development and coding, with minimal or no planning either where requirements are either unknown or the release date is not given. This model is best suited for small size projects with small development teams which are working together.

## Step 1 - Installing Apache and Updating the Firewall

- Update Ubuntu

      $sudo apt update
      
- Install Apache2 using the code

      $ sudo apt update
      #run apache2 package installation
      $ sudo apt install apache2
      Verify apache2 is running as a service




- Check how server can be accessed locally by using curl http://llocalhost:80


- Test how Apache HTTP server can respond to requests from the Internet. 


##  Step 2 — Installing MySQL

- Install MySQL 
  
      $sudo apt install mysql-server


- Run a security script that comes pre-installed with MySQL tol remove some insecure default settings and lock down access to your database system 

      $sudo mysql_secure_installation. 


*If you enabled password validation, you’ll be shown the password strength for the root password you just entered and your server will ask if you want to continue with that password. If you are happy with your current password, enter Y for “yes” at the prompt:*


- Test log in to the MySQL console 
		
      $sudo mysql


### Step 3 — Installing PHP

- Install php - process code to display dynamic content to the end user, php-mysql - a PHP module that allows PHP to communicate with MySQL-based databases and libapache2-mod-php - to enable Apache to handle PHP files. at once, run:

      $sudo apt install php php-mysql libapache2-mod-php

- Run the `$php -v`  to confirm your PHP version:

 
### Step 4 — Creating a Virtual Host for your Website using Apache

- Create the directory for projectlamp using `mkdir` command as follows:



- Assign ownership of the directory with the $USER environment variable, which will reference the current system user:



- Create and open a new configuration file in Apache’s sites-available directory using  vi :



-  Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text:

        <VirtualHost *:80>
            ServerName:projectlamp
            ServerAlias:www.projectlamp
            ServerAdmin:webmaster@localhost
            DocumentRoot: /var/www/projectlamp
            ErrorLog:${APACHE_LOG_DIR}/error.log
            CustomLog:${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

- Use `ls` command to show the new file in the sites-available directory


- Use `a2ensite` command to enable the new virtual host:


- Disable the default website that comes installed with Apache using `a2dissite` command , type

Go to your browser and open the website URL using IP address:


### Step 5 — Enable PHP on the website

- Edit the `/etc/apache2/mods-enabled/dir.conf` file and change the order in which the `index.php` file is listed within the `DirectoryIndex` directive:



- Create a new file named `index.php` inside the custom web root folder, Save and close the file then refresh the page and you will see this page:

- To remove the php file created as it contains sensitive information about the PHP environment -and  Ubuntu server Use rm to do so:



##  BLOCKER
1.  The Ubuntu server default landing page was not launching on entering the public IP address on the web browser.

**SOLUTION:** Enabled the Apache (80) in the firewall settings using $sudo ufw allow apache

2.  Unable to launch the Custom ProjectLamp landing page after disabling the apache default landing page

**SOLUTION:** Ensured the correct command `sudo a2dissite 000-default.conf` instead of `sudo a2dissite 000-default`

3.  Unable to launch the Custom ProjectLamp landing page after enabling the ProjectLamp landing page
  
**SOLUTION:** Ensured the correct command `sudo a2ensite projectlamp.conf` instead of `sudo a2ensite projectlamp`


## RESOURCES
1.  https://phoenixnap.com/blog/software-development-life-cycle#:~:text=Software%20Development%20Life%20Cycle%20is,%2C%20Test%2C%20Deploy%2C%20Maintain.
2.  https://www.tutorialspoint.com/sdlc/sdlc_overview.htm
3.  https://www.guru99.com/software-development-life-cycle-tutorial.html
4.  https://darey.io
