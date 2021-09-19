# Deploying Kubernetes Using Azure Kubernetes Service (AKS) 

This projects involves deploying a multicontainer Node.js application to Kubernetes on Azure Kubernetes Service (AKS). The application consists of two components: the API and the front end. Data from the API is stored in a MongoDB database

## Application Architecture

![image](https://user-images.githubusercontent.com/30922643/133923588-bad4c74a-6139-4650-84e6-eb11e8c95ac0.png)

## Tasks
    1. Create an Azure Kubernetes Service cluster
    2. Setup Azure Container Registry to store application container image
    3. Deploy the application components 
        - Restful API using the deployment Manifest
        - MongoDB database using Helm
        - Website Frontend using deployment manifest 
    4. Deploy Azure Kubernetes ingress using Helm 3.
    5. Expose Pods to internal and external network users
    6. Configure SSL/TLS for Azure Kubernetes Service ingress using cert-manager
    7. Monitor the health of an Azure Kubernetes Service cluster
    8. Setup horizontal pod autoscaler and cluster autoscaler for the Cluster

