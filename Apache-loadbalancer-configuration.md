# Load Balancer Solution With Apache

## Task

Deploy and configure an Apache Load Balancer for Tooling Website solution building on project 7 architecture on a separate Ubuntu EC2 instance.

## Prerequisites
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server


## Configuration of Apache As A Load Balancer
* Create an Ubuntu Server 20.04 EC2 instance
* Open TCP port 80 on by creating an Inbound Rule in Security Group on AWS
* Install Apache Load Balancer on The server and configure it to point traffic coming to LB to both Web Servers:

      sudo apt update
      sudo apt install apache2 -y
      sudo apt-get install libxml2-dev

![image](https://user-images.githubusercontent.com/30922643/122263222-18d30700-cece-11eb-97a2-52f8b4716a31.png)

![image](https://user-images.githubusercontent.com/30922643/122263247-1ffa1500-cece-11eb-984b-cc18d1f26482.png)


- Enable following modules:

      sudo a2enmod rewrite
      sudo a2enmod proxy
      sudo a2enmod proxy_balancer
      sudo a2enmod proxy_http
      sudo a2enmod headers
      sudo a2enmod lbmethod_bytraffic

- Restart apache2 service

      sudo systemctl restart apache2

- Check the status of Apache2 to make sure it is up and running 
      
      sudo systemctl status apache2

![image](https://user-images.githubusercontent.com/30922643/122263287-2a1c1380-cece-11eb-9d21-2a2a943de905.png)

- Configure the load balancer as follows
    
      sudo vi /etc/apache2/sites-available/000-default.conf

      
- Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

      <Proxy "balancer://mycluster">
                     BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
                     BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
                     ProxySet lbmethod=bytraffic
                     # ProxySet lbmethod=byrequests
              </Proxy>

              ProxyPreserveHost On
              ProxyPass / balancer://mycluster/
              ProxyPassReverse / balancer://mycluster/

 - Restart apache server

        sudo systemctl restart apache2

![image](https://user-images.githubusercontent.com/30922643/122263310-30aa8b00-cece-11eb-9170-fa45c67b93ab.png)

* Try other methods, like: bybusyness, byrequests, heartbeat


* Try to access the Load balacer’s public IP address or Public DNS name using a browser to verify the configuration is correct:

![image](https://user-images.githubusercontent.com/30922643/122263328-37d19900-cece-11eb-99ac-3b4a277f3b81.png)

**Optional Step - Configuration of Local DNS Names Resolution**

* Configure a local domain name resolution by updating /etc/hosts file
* Open /etc/hosts on the load balancer server.            
    
      sudo vi /etc/hosts
* Add 2 records into this file with Local IP address and arbitrary name for both of the Web Servers <WebServer1-Private-IP-Address>Web1 <WebServer2-Private-IP-Address> Web2

![image](https://user-images.githubusercontent.com/30922643/122263350-3f913d80-cece-11eb-8249-e099879ab471.png)


* Update the Load Balancer configuration file with the names instead of IP addresses.

![image](https://user-images.githubusercontent.com/30922643/122263384-46b84b80-cece-11eb-938e-449cee147656.png)

* Try to curl the Web Servers from LOad balancer locally using curl http://Web1 or curl http://Web2 - it shall work.     Note: this is only internal configuration and it is also local to the LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

![image](https://user-images.githubusercontent.com/30922643/122263406-4cae2c80-cece-11eb-969c-c6f8672ad7a0.png)

## BLOCKERS
1. The implementation of this project was quite easy. I did not encounter any technical challenge.
2. The only blocker I had was having to switch to using a mobile device to complete the project due to some faulty parts of my laptop and this was really not flexible enough to implement all the extra tasks.

## RESOURCES
1. https://darey.io
2. https://stackoverflow.com/questions/10494431/sticky-and-non-sticky-sessions
3. https://access.redhat.com/solutions/90093
4. https://www.nginx.com/resources/glossary/layer-4-load-balancing/
5. https://www.nginx.com/resources/glossary/load-balancing/
6. https://www.f5.com/services/resources/glossary/load-balancer
7. https://www.a10networks.com/blog/how-do-layer-4-and-layer-7-load-balancing-differ/
8. https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html



