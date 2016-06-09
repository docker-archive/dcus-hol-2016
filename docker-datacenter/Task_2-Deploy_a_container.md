# Task 2: Deploy a container

In this task you will use USP to deploy a web server from the official NGINX image. The following steps will walk you through this process.

1. Deploy a container
2. Test the deployment

## Prerequisites
- UCP setup with 3 nodes connected to the UCP controller

## Step 1 - Deploy a container
In this step you will launch a new container based on the NGINX image using the UCP web UI.

1. If you have not already done so, log in to UCP with the built-in **admin** account.

2. Click the **Containers** link on left navigation bar.

3. Click on **+ Deploy Container** button.

4. Fill out the Basic Settings as shown below:

  ![](http://i.imgur.com/6upe6pQ.png)

5. Expand the **Network** section on the same page and configure the following port mappings:

  ![](http://i.imgur.com/yazdZZf.png)

6. Hit the **Run Container** button on the right side panel.

  When the operation completes you will see your container listed as shown below. The green circle to the left of the container indicates that the container is in the **running** state.

  ![](http://i.imgur.com/7DorWT8.png)

7. Click on the row where the container is listed to see the full container details. Then scroll down to the **Ports** section of the page to check the port mappings.

  ![](http://i.imgur.com/vQGDHGE.png)

## Step 2 - Quick Test

In this step you will use your web browser to access the home page of the **nginx_server** container started in the previous step.

In order to access the NGINX container from your web browser you will need the DNS hostname of the node that the container is running on.

1. First, let's take a look at the node our **nginx_server** container is running on. In the container details, you can find the node information.

   ![](http://i.imgur.com/gHMO7O8.png)

   In this particular example, the **nginx_server** container is running on the **node--1** node with an IP of 10.0.18.236. However, this is the private IP address of the node and you will not be able to use this address to connect to the web server. Locate the public IP, or public DNS name, of the node from the lab details you received (each lab machine you have will have a public and priave IP and DNS).

2. Go to your web browser and enter the public IP or public DNS name of the node that the **nginx_server** container is running on.

   You will see the NGINX welcome page.

   ![](http://i.imgur.com/W8wWNH9.png)

You have successfully launched a web container using the Docker UCP web UI.