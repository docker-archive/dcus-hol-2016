# Orchestration with Swarm Computing

> **Difficulty**: Beginner

> **Time**: Approximately 40 minutes

In this lab you will deploy a Dockerized application to a single host and test the application. You will then configure Docker for Swarm Computing and deploy the same app across multiple hosts. You will then see how to scale the application and recover from host failures.

You will complete the following steps in this lab:

- [Deploy a single host application](#deploy-application)
- [Configure Docker for Swarm Computing](#start-cluster)
- [Deploy the application across multiple hosts](#multi-application)
- [Scale the application](#scale-application)
- [Recover from a host failure](#recover-application)

## Pre-requisites

You will need all of the following in order to complete this lab:
- Three nodes running Docker v1.12.x. You will be using **v112node0**, **v112node1**, and **v112node2**.
- No containers running on these three nodes - `docker rm -f $(docker ps -aq`)
- A Docker ID. Creating a Docker ID is free, and allows you to push and pull images from Docker Hub. [This link](https://docs.docker.com/mac/step_five/) describes how to create a Docker ID (you only need to complete the procedure up to step 2.3).

> **NOTE TO MARK: DO WE HAVE DETAILS OF HOW USERS WILL KNOW THEIR LAB DETAILS - DNS names, IP addresses etc?

## <a name="deploy-application"></a>Step 1: Deploy a single host application

**MARK: In other labs I've reviewed I've referred to sections like these as "Steps". "Step 1 - Blah blah blah...".  Because other labs have been large and broken into multiple docs - each doc called a *Task*. So I've renamed your tasks here as steps.**

In this step you will deploy a simple application that runs on a single Docker host. In order to do that, you will complete the following:

- Clone the app's GitHub repo
- Dockerize the app
- Push the image to Docker Hub
- Run the app

The application you will deploy is the **FoodTrucks** application. It is a 2-container application comprised of a web front end and a database backend. The frontend is a custom Python application and the backend is Elasticsearch. It creates a great visualization to see all of the different places to get food in San Francisco. The locations are searchable and presented on a map of the city.

### Step 1.1 - Clone the app's GitHub repo

1. SSH to your __v112node0__.

   The command to SSH into **v112node0** will look something like the following:

   ```
   ssh <username>@<public-dns-of-v112node0>
   ```

   **MARK: If the labs are in AWS the command will look like the following. I'm gonna assume AWS for the remainder**

   ```
   $ ssh -i <private-key-file> ubuntu@<public-dns-of-v112node0>
   ```

   Be sure to substitute your own private key file and your **v112node0** public DNS name in the command above.

2. Use `git` to clone the app's GitHub repo.

   ```bash
   ubuntu@v112node0:~/ $ git clone https://github.com/mark-church/FoodTrucks.git
Cloning into 'FoodTrucks'...
remote: Counting objects: 357, done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 357 (delta 10), reused 0 (delta 0), pack-reused 327
Receiving objects: 100% (357/357), 2.86 MiB | 2.51 MiB/s, done.
Resolving deltas: 100% (133/133), done.
Checking connectivity... done.
   ```
   The repo contains all of the files and code that is required to run the web front-end portion of the app. The command above copies (clones) the repo into a new directory on your machine called `FoodTrucks`.

3. Change directory to `FoodTrucks` and examine the list of files.

   ```bash
   ubuntu@v112node0:~/$ cd FoodTrucks

   ubuntu@v112node0:~/FoodTrucks/$ tree -L 2
   .
   ├── Dockerfile
   ├── README.md
   └── app
       ├── app.py
       ├── package.json
       ├── requirements.txt
       ├── static
       ├── templates
       └── webpack.config.js

   3 directories, 6 files
   ```

   If `tree` is not installed, either use the `ls` command instead, or install `tree` with the `apt-get install tree -y` command.

   Some of the files worth knowing include the following:

 -   __Dockerfile__: This file contains the recipe for the web front-end component of the `FoodTrucks` app.

  - __app.py__: This is the main Python module that serves content.

   - **/app/templates**: This folder includes the HTML web page that is called by the Python web server.

   Let's focus on the **Dockerfile** for a second. A **Dockerfile** is a text file that contains all the **instructions** required to build an application or service into a Docker image. This includes instructions to install packages, copy data, insert metadata, and anything else that should be included as part of the image. The Docker Engine uses Dockerfiles to create new images.

### Step 1.2 -  Dockerize the app

You **Dockerize** an application by describing it in a **Dockerfile** and using that **Dockerfile** to create a Docker image.

The following procedure will guide you through Dockerizing the web front-end portion of the `FoodTrucks` app. The Elasticsearch back-end component of the app is already Dockerized and available on Docker Hub as an image.

Make sure you're logged in to **v112node0** and in the `~/FoodTrucks/` directory.

1. Verify that the Docker is running.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker version
   Client:
    Version:      1.12.0-dev
    API version:  1.24
    Go version:   go1.6.2
    Git commit:   5ae6d21
    Built:        Wed Jun  8 06:21:03 2016
    OS/Arch:      linux/amd64

   Server:
    Version:      1.12.0-dev
    API version:  1.24
    Go version:   go1.6.2
    Git commit:   5ae6d21
    Built:        Wed Jun  8 06:21:03 2016
    OS/Arch:      linux/amd64
   ```

4. Inspect the contents of the Dockerfile.

   ```bash
   ubuntu@v112node0:~/FoodTrucks/$ cat Dockerfile
   FROM ubuntu:14.04

   RUN apt-get -yqq update
   RUN apt-get -yqq install python-pip python-dev nodejs npm
   RUN ln -s /usr/bin/nodejs /usr/bin/node

   ADD app /opt/flask-app
   WORKDIR /opt/flask-app

   RUN npm install
   RUN npm run build
   RUN pip install -r requirements.txt

   EXPOSE 5000

   CMD [ "python", "./app.py" ]
   ```

   Let's have a quick look at the contents of the Dockerfile.

   __FROM__: The FROM instruction sets the base image that all subsequent instructions will build on top of. The base image can be any valid Docker image including other peoples' images. In this exercise you are starting with the `ubuntu:14.04` image as your base image.

   __RUN__: The RUN instruction executes commands while the image is being built. Each RUN instruction creates a new image layer. The RUN instructions in this **Dockerfile** are updating the local `apt` package lists from source, installing some packages, creating a link, and running some `npm` commands.

   __ADD__: The ADD instruction copies files or directories from the Docker host and adds them to the filesystem of the container. In this particular **Dockerfile** the `app` directory that was just cloned to **v112node0** form GitHub is copied to the image at `/opt/flask-app`.

   __EXPOSE__: The EXPOSE instruction lists the ports that the container will expose at runtime. This image will expose port 5000.

   __CMD__: The main purpose of CMD instruction is to provide defaults for an executing container. This CMD instruction will run `python` specifying `./app.py` as the argument.

5. Use the following `docker build` command to build an image from the Dockerfile.

   Be sure to substitute **markchurch** in this example with your own Docker ID.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker build -t markchurch/foodtruck-web .
   ```

   It may take a minute or two to build the image. This is because the `ubuntu:14.04` image has to be pulled locally, and several packages have to be installed into the image.

   The `-t` flag lets you **tag** the image. The `<your-docker-id>/foodtruck-web` will be the image tag - **Be sure to substitute your Docker ID**. Tagging the image lets you easily identify it as well as push to Docker Hub or other Docker Registries.

   The trailing period `.` tells the command to send your current working directory to the Docker daemon as the build context.  The **build context** is a fancy way of saying the Dockerfile and other files required to build the image.

   The output of the `docker build` command will look similar to the following (some lines have been removed for brevity).

   >**Warning**: You may see red text scroll up the screen as part of the build. This is expected behavior.

   ```
   Sending build context to Docker daemon 46.37 MB
   Step 1 : FROM ubuntu:14.04
   <Snip>
    ---> 8f1bd21bd25c
   Step 2 : RUN apt-get -yqq update
    <Snip>
    ---> 8ff670592378
   Step 3 : RUN apt-get -yqq install python-pip python-dev nodejs npm
    <Snip>
    ---> 308b2a7f054e
   Step 4 : RUN ln -s /usr/bin/nodejs /usr/bin/node
    <Snip>
    ---> 410bcf478b3c
   Step 5 : ADD app /opt/flask-app
    <Snip>
    ---> 15494eeae7df
   Step 6 : WORKDIR /opt/flask-app
    <Snip>
    ---> d9e54bd739df
   Step 7 : RUN npm install
    <Snip>
    ---> 09f236a1b28a
   Step 8 : RUN npm run build
    <Snip>
    ---> de45aac30190
   Step 9 : RUN pip install -r requirements.txt
    <Snip>
    ---> aeae46ed4044
   Step 10 : EXPOSE 5000
    <Snip>
    ---> ac1faa1e23ea
   Step 11 : CMD python ./app.py
    <Snip>
    ---> e015bed6be19
   Successfully built e015bed6be19
   ```

   Your output might be a lot more verbose if this is the first time you have pulled the `ubuntu:14.04` image and ran the `docker build` command.

6. Run a `docker images` command to confirm that the image is listed.

   ```
   $ docker images
   REPOSITORY                 TAG       IMAGE ID        CREATED          SIZE
   markchurch/foodtruck-web   latest    e20087618c79    3 minutes ago    589.1 MB
   ```

   The output of your command will show your Docker ID and not *markchurch*. You will also see the `ubuntu:14.04` image listed.

Congratulations. You have successfully Dockerized the web front-end component of the **FoodTrucks** app.  

### Step 1.3 - Push the image to Docker Hub

In this section you will push the newly created image to Docker Hub so that you can pull it form other nodes.  

Perform the following steps from **v112node0**.

1. Login with your Docker ID.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker login
   Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
   Username: <your-docker-id>
   Password:
   Login Succeeded
   ```

2. Push your image to Docker Hub. Later in the lab you will be pulling this image to different hosts.

   Remember to substitute your Docker ID.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker push markchurch/foodtruck-web
   The push refers to a repository [docker.io/markchurch/foodtruck-web]
   ef367edd68c8: Pushed
   65c950fa7c3b: Pushed
   ecefedce4aca: Pushed
   b9acf9e4b1b0: Pushed
   909f6ca0a2ac: Pushed
   22830451f3d8: Pushed
   cbadaa15a015: Pushed
   5f70bf18a086: Pushed
   6f8be37bd578: Pushed
   9f7ab087e6e6: Pushed
   dc109d4b4ccf: Pushed
   a7e1c363defb: Pushed
   latest: digest: sha256:57013bc3a8a99665e08b7576eda542b6dee938841807e8986f55fddb10799b8f size: 2832
   ```

   This will push your newly built image to your own public repository on Docker Hub called `<your-docker-id>/foodtruck-web`.

   Docker Hub is the default Docker Registry and is hosted on the public internet. You can also push images to Docker Trusted Registry as well as third party registries.

### Step 1.4 - Run the App

Now that you've Dockerized the web front-end for the app (the backend Elasticsearch is already Docckerized) it's time to deploy the app. To do that you will:

- Create a new network
- Deploy the Elasticsearch back-end
- Deploy the web front-end

Perform all of these steps from **v112node0**.

1. Create a new network for the containers in the app to use.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker network create foodnet
   6c794a6c5129a360b65e384d7a92dc0758c81048ce4c1068def53653233c
   ```

   This will create a new *bridge* network for the containers in your app to use. This will allow the containers in your app to comminucate with each other. Containers on other networks will not be able to communicate with them.

2. Deploy the Elasticsearch component of the app on the `foodnet` network and name is `es`.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker run -d --name es --net foodnet elasticsearch
   Unable to find image 'elasticsearch:latest' locally
   latest: Pulling from library/elasticsearch
   51f5c6a04d83: Pull complete
   65e9ddd8bd7a: Pull complete
   02500df954bf: Pull complete
   e3b067df5fd6: Pull complete
   121bc2f7d35c: Pull complete
   33b88e5aef8c: Pull complete
   9e022032b28e: Pull complete
   bcc87244e8c5: Pull complete
   da8b8f2cc010: Pull complete
   c7f16c967766: Pull complete
   82dcdf2d00af: Pull complete
   7548dc5b4abd: Pull complete
   147cbdfeb7f2: Pull complete
   bb1d984d90eb: Pull complete
   Digest: sha256:3885c25b9f05ac7de7e81f9f82ddf25c909e8de70cb15cd76b0c00058821a718
   Status: Downloaded newer image for elasticsearch:latest
   49c901f8cecbd933302fac2fa535113c0803a5f42d03525305602cbc640ae869
   ```

   The `-d` flag tells Docker to run the `es` container in the background and not attach it to your local terminal.

3. Deploy the web front-end component of the app on the `foodnet` network and expose port 80 so that the application can be accessed on port 80.

   Remember to substitute your Docker ID.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker run -d --name web -p 80:5000 --net foodnet markchurch/foodtruck-web
   c2ac7b80eaa6fc2759e80d40a3382277bd437b0294816f41f22130f11de16b3d
   ```

4. Check that both containers are running.

   ```
   $ docker ps
   CONTAINER ID    IMAGE                       COMMAND                  CREATED              STATUS              PORTS                    NAMES
   c2ac7b80eaa6    markchurch/foodtruck-web    "python ./app.py"        9 seconds ago        Up 8 seconds        0.0.0.0:80->5000/tcp     web
   49c901f8cecb    elasticsearch               "/docker-entrypoint.s"   About a minute ago   Up About a minute   9200/tcp, 9300/tcp       es
   ```

   The output above shows that both containers are up and running. You can see that one is based on the `elasticsearch` image and the other on the `<you-docker-id>/foodtruck-web` image. You can also see that port 80 on the Docker host is mapped to port 5000 on the web front-end container.

5. Point your web browser to the app.

   Paste the public DNS or public IP of **v112node0** into your web browser.

   You should see the following ...
   <p align="center">
   <img src="images/food.png" width=400px>
   </p>

Congratulations. You have laucnhed your newly Dockerized webb app.

Before proceeding to the next task, clear all of the containers on __v112node0__ by running the following command.

```
ubuntu@v112node0:~/FoodTrucks/$ docker rm -f $(docker ps -q)
```

Verify that the command worked and there are no running or stopped containers on the host.

```
ubuntu@v112node0:~/FoodTrucks/$ docker ps -a
```

## <a name="start-cluster"></a>Step 2: Configure Swarm Mode

So far you have deployed an application to a single Docker host (node). However, real-world applications are typically deployed across multiple hosts. This improves application performance and availability, as well as allowing individual application components to scale independently. Docker has powerful native tools to help you do this.

In this step you will configure *Swarm Mode*. This is a new optional mode in which multiple Docker Engines form into a self-orchestrating group of engines called a *swarm*. Swarm mode enables new features such as *services* and *bundles* that help you deploy and manage multi-container apps across multiple Docker hosts.

You will complete the following:

- Configure *Swarm mode*
- Run the app
- Scale the app
- Recover from failed nodes

For the remainder of this lab we will refer to *Swarm Mode (native Docker clustering)* as ***Swarm mode***. The collection of Docker engines configured for Swarm mode will be referred to as the *swarm*.

A swarm comprises one or more *Manager Nodes* and one or more *Worker Nodes*. The manager nodes maintain the state of swarm and schedule appication containers. The worker nodes run the application containers. As of Docker 1.12, no external backend, or 3rd party components, are required for a fully functioning swarm - everything is built-in!

In this part of the demo you will use all three of the nodes in your lab. __v112node0__ will be the Swarm manager, while __v112node1__ and __v112node2__ will be worker nodes. Swarm mode supports a highly available redundant manager nodes, but for the purposes of this lab you will only deploy a single manager node.

#### Step 2.1 - Create a Manager node

1. If you haven't already done so, SSH in to **v112node0**.

   For example (remember to substitute your SSH key and **v112node0** for your lab):

   ```
   $ ssh -i mark.pem ubuntu@ec2-54-171-183-76.eu-west-1.compute.amazonaws.com
   ```

2. Get the internal/private IP address of __v112node0__.

   ```bash
   ubuntu@v112node0:~/FoodTrucks/$ ifconfig eth0
   eth0      Link encap:Ethernet  HWaddr 06:67:be:89:1f:b5
             inet addr:172.31.22.238  Bcast:172.31.31.255  Mask:255.255.240.0
   ...
   ```

   In the example above, the internal IP address is **172.31.22.238** as indicated by the `inet addr` field.

   Make a note of this IP address as you will use it later when adding worker nodes to the swarm.

3. Create a Manager node on __v112node0__ using its internal IP address.

   Remember to substitute the IP address for the private IP of your **v112node0**.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker swarm init --listen-addr 172.31.22.238:2377
   Swarm initialized: current node (0gohc21qtm7sp) is now a manager.
   ```

   `docker swarm init` is a new command in Docker 1.12. It is entirely optional, but is all that is needed to initialize a new *swarm* (native Docker cluster).

   Port `2377` is recommended but not mandatory. You can use a different port of your choosing.

4. Run a `docker info` command to verify that **v112node0** was successfully configured as a swarm manager node.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker info
   Containers: 0
   Running: 0
   <Snip>
Swarm:
 NodeID: 3vatl908ksmjk
 IsManager: YES
<Snip>
   ```

The swarm is now initialized with **v112node0** as the only Manager node. In the next section you will add **v112node1** and **v112node2** as *Worker nodes*.

####  Step 2.2 - Join Worker nodes to the Swarm

You will perform the following procedure on **v112node1** and **v112node2**. Towards the end of the procedure you will switch to **v112node0**.

1. Open a new SSH session to __v112node1__ (Keep your SSH session to **v112node0** open in another tab or window).

   ```
   $ ssh -i <your-ssh-key> ubuntu@<v112node1-public-ip-or-dns>
   ```

2. Join the swarm using the *internal* IP of **v112node0** and the port specified when you created the Swarm.

   The format of the command is as follows:

   ```
   docker swarm join <internal-ip-of-manager-node>:<port>
   ```

   The example below joins a new worker node to an existing swarm with a manager node with IP address of 172.31.22.238 on port 2377.

   Be sure to substitute the internal IP address of your **v112node0** that you made a note of earlier.

   ```
   $ docker swarm join 172.31.22.238:2377
   This node joined a Swarm as a worker.
   ```

3. Repeat steps 1 and 2 for __v112node2__.

4. Switch to **v112node0** and verify that the Swarm is up and running with **v112node0** as the Manager node and **v112node1** and **v112node2** both as Worker nodes.

   Run this command on **v112node0** (the Manager node).

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker node ls
   ID               NAME            MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS  LEADER
   0gohc21qtm7sp *  node-0          Accepted    Ready   Active        Reachable       Yes
   0v3wh1gbomf8y    node-1          Accepted    Ready   Active
   151wlv08pt7aq    node-2          Accepted    Ready   Active
   ```

   The `docker node ls` command shows you all of the nodes that are in the swarm as well as their roles in the swarm. The `*` identifies the node that you are issuing the command from.

Congratulations. You have configured a swarm with one manager node and two worker nodes.

## <a name="multi-application"></a>Step 3: Deploy a Multi-Service, Multi-Host Application

Now that you have a swarm up and running, it is time to deploy the app to it.  To do this you will complete the following actions:

- Create a new overlay network for the application
- Deploy the application components as Docker *services*

You will perform the following procedure from **v112node0**.

### Step 3.1 - Create a new overlay network for the application

1. Create a new overlay network called `ovnet`.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker network create -d overlay ovnet
   et6r5n0cz6coykgplzymwd01n
   ```

   The `-d` flag let's you specify which network driver to use. In this example you are telling Docker to create a new network using the **overlay** driver.

2. Confirm that the network was created and inspect its configuration.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker network ls
   NETWORK ID          NAME                DRIVER
   27a561be6a03        bridge              bridge
   5eb4f6d1f4ff        docker_gwbridge     bridge
   8d71645fa144        host                host
   5x9zxmtax6js        ingress             overlay
   5ea19b763513        none                null
   eupq9o68egub        ovnet               overlay

   ubuntu@v112node0:~/FoodTrucks/$ docker network inspect ovnet
   [
    {
        "Name": "ovnet",
        "Id": "eupq9o68egubof077xr31luzb",
        "Scope": "global",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "257"
        },
        "Labels": null
    }
]
   ```

Now that the container network is created you are ready to deploy the application to the swarm.

#### Step 3.2 - Deploy the application components as Docker services

*Services* are a new concept in Docker 1.12. They work with swarms and are intended for long-running containers based on identical images.

In this section you will deploy two services: one for the Elasticsearch backend and one for the web front-end.

You will perform this procedure from **v112node0**.

1. Deploy a single Elasticsearch container as part of the `es` *service*.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service create --scale 1 --name es --network ovnet elasticsearch
   1wpdd1qnv86k4ahpe63osbp7i
   ```

  The `--scale` option allows you to specify how many containers should be part of the service. You will use this option again later to scale the service up and down.

2. Verify the service is up.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service ls
   ID            NAME  SCALE  IMAGE          COMMAND
   1wpdd1qnv86k  es    1      elasticsearch
   ```

3. Deploy the web front-end using the `foodtruck-web` image you created earlier and expose it on port 80.

   Remember to substitute `markchurch` with your own Docker ID.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service create --scale 1 --name web --network ovnet -p 80:5000 markchurch/foodtruck-web
   br6bnsntlh0b0j49074p7fwr2
   ```

4. Run another `docker service ls` to make sure both services are up and running.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service ls
   ID            NAME  SCALE  IMAGE                     COMMAND
   1wpdd1qnv86k  es    1      elasticsearch
   br6bnsntlh0b  web   1      markchurch/foodtruck-web
   ```

5. Inspect each service more closely.

   You can look more closely at each service with the following commands:

   - `docker service inspect <service-name-or-service-id>`
   - `docker service tasks <service-name-or-service-id>`

6. Check the app with your web browser.

   Point your web browser to `http://<v112node0-public-ip>`

   <NEED IMAGE>

   You can point your web browser to the public IP or public DNS of any node in the swarm.

Well done. You have deployed the FoodTruck app to your new Swarm using Docker services.

#### Step 3.3 - Scale the app

One of the great things about *services* is that you can scale them up and down to meet demand. In this step you'll scale the web front-end service up and then back down.

You will perform the following procedure from **v112node0**.

1. Scale the number of containers in the **web** service to 5.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service update --scale 5 web
   web
   ```

2. Confirm the scaling operation using the command line.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service tasks web
   dzyehfy4lyfr7gq2pq6wmgsg6  web.1  web      markchurch/foodtruck-web  RUNNING 43 seconds    RUNNING        node--2
   8qvzomj4cymp9b3lgo2oz7ttk  web.2  web      markchurch/foodtruck-web  RUNNING 53 seconds    RUNNING        node--2
   5mri49laja1bem4tdwvxtj7t8  web.3  web      markchurch/foodtruck-web  ACCEPTED 54 seconds   RUNNING        node--1
   bxfzx8xvv617ntxebpikclczx  web.4  web      markchurch/foodtruck-web  PREPARING 54 seconds  RUNNING        node--1
  9p1bc4y1g3zygroh6474qnkvu  web.5  web      markchurch/foodtruck-web  RUNNING 53 seconds    RUNNING        node--2
   ```

   Notice that there are now 5 containers listed. It may take a few seconds for the new containers in the service to all show as **RUNNING**.

3. Confirm the scaling operation and container load-balancing from a web browser.

   Point your web browser to `http://<any-node-ip>` and hit the `refresh` button a few times. You will see the container ID and internal IP changing. This is because the swarm is automatically load balancing across all five containers.

4. Scale the service back down just two containers.

   ```
   ubuntu@v112node0:~/FoodTrucks/$ docker service update --scale 2 web
   web
   ```

5. Verify that the number of containers has been reduced to 2 using the `docker service tasks web` command and your web browser.

You have successfully scaled a swarm service up and down.

Before moving on to the next step, scale the `web` service back up to 5. This will help you better visualise what is being demonstrated.

```
ubuntu@v112node0:~/FoodTrucks/$ docker service update --scale 5 web
```

#### Step 3.4 - Recover from a node failure

Now let's simulate a worker node failure and see how the swarm deals with it.

1. SSH in to **v112node2** and stop the Docker daemon.  

   ```bash
   ubuntu@v112node2:~/$ service docker stop
   docker stop/waiting
   ```

   **MARK: I'd prefer to be able to *STOP* the ec2 instance but doubt the lab users will have access to do that (AWS console etc.....).  I've tested this and it works.**

2. Switch back to **v112node0** and check the status of the containers in the service.

   ```bash
   ubuntu@v112node0:~/FoodTrucks/$ docker service tasks web
   4f4p6ysoi3vhwwsil6g02yys8  web.1  web      markchurch/foodtruck-web  RUNNING 43 seconds    RUNNING        node--1
   5jmnhi62478hytaebfeeql7pj  web.2  web      markchurch/foodtruck-web  RUNNING 53 seconds    RUNNING        node--1
   5mri49laja1bem4tdwvxtj7t8  web.3  web      markchurch/foodtruck-web  RUNNING 5 minutes     RUNNING        node--1
   bxfzx8xvv617ntxebpikclczx  web.4  web      markchurch/foodtruck-web  RUNNING 5 minutes     RUNNING        node--1
   99mu5nc2hfho98x3ejhxniu7e  web.5  web      markchurch/foodtruck-web  RUNNING 53 seconds    RUNNING        node--1
   ```

   Note that all of the web containers are now running on only one of the two worker nodes. **v112node2** no longer has any containers running on it.

3. Check the status of the nodes in the swarm.

   ```
   $ docker node ls
   ID               NAME     MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS  LEADER
   0mu982nw4lqpu *  node--0  Accepted    Ready   Active        Reachable       Yes
   22zbgwlvixvft    node--1  Accepted    Ready   Active
   22zbgwlvixvft    node--2  Accepted    Down    Active

   ```

   **node-2** is showing as **down** because the Docker daemon on the node is in the stopped state.

Congratulations! You've completed this lab. You now know how to build a swarm, deploy applications as collections of services, and scale individual services up and down.
