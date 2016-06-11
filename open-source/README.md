# Orchestration with Docker Clustering

> **Difficulty**: Beginner

> **Time**: Approximately 40 minutes

> **Notes**
>
> * In this lab you will be using **node-0**, **node-1**, and **node-2**
> * Ensure that no containers are running on these nodes ```$ docker rm -f $(docker ps -q)```
> * This lab requires a Docker Hub account. This account is free and will allow you to push and pull images from the Docker public registry. This link describes how to create a Docker Hub account: <a href="https://docs.docker.com/mac/step_five/">https://docs.docker.com/mac/step_five/</a>

> **Tasks**:
>
> * [Deploy a Simple Application](#deploy-application)
> * [Start a Docker Cluster](#start-cluster)
> * [Deploy a Multi-Service, Multi-Host Application](#multi-application)

## <a name="deploy-application"></a>Task 1: Deploy an Application on a Single Node

### Set Up Environment

**Step 1:** Connect to **node-0**. (How will the users connect to nodes??)

**Step 2:** Use the following command to clone the FoodTrucks repo from GitHub to node-0. You can see this repo for yourself by going to https://github.com/mark-church/FoodTrucks.

```
node-0:$ git clone https://github.com/mark-church/FoodTrucks.git
Cloning into 'FoodTrucks'...
remote: Counting objects: 357, done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 357 (delta 10), reused 0 (delta 0), pack-reused 327
Receiving objects: 100% (357/357), 2.86 MiB | 2.51 MiB/s, done.
Resolving deltas: 100% (133/133), done.
Checking connectivity... done.
```

**Step 3:** Change directory to `FoodTrucks` and examine the list of files in the repo by using `tree -L 2`

```
$ cd FoodTrucks
$ tree -L 2
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

**FoodTrucks** is a simple 2 container application. It's comprised of a web front end and a database backend. The frontend is a custom Python application and the backend is Elasticsearch. It creates a great visualization to see all of the different places to get food in San Francisco. The locations are searchable and presented on a map of the city. 

We can see a number of files here.

**Dockerfile** - this file is the recipe for the web front-end of the FoodTrucks app

**app.py** - is the main python module that serves the content

**templates** - is a folder that includes the HTML web page that's called by our python web-server


Let's focus on the **Dockerfile** for a second. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. This includes packages that the application will need, directories of data that needs to be a part of the image, and any metadata that should be shipped in the image. The Docker engine uses Dockerfiles to create new container images.

Next we will use a Dockerfile to create an image and then run a container from that image.

### Build the Image

**Step 1:** Verify that the Docker is running on **node-0** with `docker version`

```
$ docker version
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

**Step 2:** Now inspect the Dockerfile with `cat Dockerfile` to see how its parameters will build the container image.

```
$ cat Dockerfile
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

**FROM**: This sets the base image for subsequent instructions. This can be any valid image. You can also build new images from other peoples' images.

**ADD**: This copies files or directories and adds them to the filesystem of the container. This copied over the contents of our working directory, and thus the app.py file.

**RUN**: The RUN instruction will execute any commands in a new layer on top of the current image and commit the results.

**EXPOSE**: This command exposes a specific port inside the container. 

**CMD**: The main purpose of a CMD is to provide defaults for an executing container.

**Step 3:** Now we're going to build a `foodtruck-web` image from the Dockerfile. We will build an image from the current directory and then tag the image with our own Dockerhub ID and the image name. Run the command `docker build -t <Your Dockerhub ID>/foodtruck-web . ` with the Dockerhub ID that you registered with. Some of the output below is removed to show the individual steps of the build process.

```
$ docker build -t markchurch/foodtruck-web .

Sending build context to Docker daemon 46.37 MB
Step 1 : FROM ubuntu:14.04
 ---> 8f1bd21bd25c
Step 2 : RUN apt-get -yqq update
 ---> Using cache
 ---> 8ff670592378
Step 3 : RUN apt-get -yqq install python-pip python-dev nodejs npm
 ---> Using cache
 ---> 308b2a7f054e
Step 4 : RUN ln -s /usr/bin/nodejs /usr/bin/node
 ---> Using cache
 ---> 410bcf478b3c
Step 5 : ADD app /opt/flask-app
 ---> Using cache
 ---> 15494eeae7df
Step 6 : WORKDIR /opt/flask-app
 ---> Using cache
 ---> d9e54bd739df
Step 7 : RUN npm install
 ---> Using cache
 ---> 09f236a1b28a
Step 8 : RUN npm run build
 ---> Using cache
 ---> de45aac30190
Step 9 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> aeae46ed4044
Step 10 : EXPOSE 5000
 ---> Using cache
 ---> ac1faa1e23ea
Step 11 : CMD python ./app.py
 ---> Using cache
 ---> e015bed6be19
Successfully built e015bed6be19
 ```
This will take roughly 2 minutes so take some time to take read up on Dockerfiles here: https://docs.docker.com/engine/reference/builder/


##### NIGEL - some comments about what each block of commands is doing in the above Dockerfile


**Step 4:** Run `docker images` and confirm that the image is listed

```
$ docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
markchurch/foodtruck-web   latest              e20087618c79        3 minutes ago       589.1 MB
```
We have now built a new container image using the Dockerfile and the contents of the directory that we cloned from GitHub.
Next we will push the image to a registry so that it can be stored and even used by other people.

### Push the Image

**Step 5:** Login with your Docker ID to push and pull images from Docker Hub with `docker login`. 

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: markchurch
Password:
Login Succeeded
```

**Step 6:** Now push your image to the Docker Hub with `docker push <Your Dockerhub ID>/foodtruck-web`. Later in this lab we will be pulling the image down from the Dockerhub to different hosts.

```
$ docker push markchurch/foodtruck-web
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

### Run the Application

Now we're going to create a containers from the `elasticsearch` and `foodtruck-web` images that will make up our application.

1. Let's create a private network for these containers to use. They will be able to communicate with eachother but containers in other networks will not be able to communicate with them.



```
$ docker network create foodnet
6c794a6c5129a360b65e384d7a92dc0758c81048ce4c1068def53653233c1ab5
```

By default the `docker network create` will create a `bridge` network and assign it an available private subnet

2. Deploy elasticsearch on the `foodnet` network and name it `es`. '-d' is used to deploy the application in the background and not connected to this terminal. Because the elasticsearch image is not already stored locally the Docker engine will pull it down from the Docker Hub.

```
$ docker run -d --name es --net foodnet elasticsearch
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

3. Deploy our `foodtruck-web` image as a container on the `foodnet` network. We will expose port 80 externally which is where our application can be accessed from the outside.

```
$ docker run -d --name web -p 80:5000 --net foodnet markchurch/foodtruck-web
c2ac7b80eaa6fc2759e80d40a3382277bd437b0294816f41f22130f11de16b3d
```

4. Now use `docker ps` to check that our containers are up and running.



```
$ docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED              STATUS              PORTS                    NAMES
c2ac7b80eaa6        markchurch/foodtruck-web        "python ./app.py"        9 seconds ago        Up 8 seconds        0.0.0.0:80->5000/tcp   web
49c901f8cecb        elasticsearch                   "/docker-entrypoint.s"   About a minute ago   Up About a minute   9200/tcp, 9300/tcp       es
```

From this output we can see the following:

* The container has been given an automatically generated ID and name (we can specify the name if we choose)
* We can see what image was used to create the container
* We can also see if and how the container is exposed outside the host (in this case on port 5000 of the host interfaces)

5. Lastly, let's use the browser to connect to our live container. Look up the public facing IP address of **node-0**. Type ```http://<ip address>``` into the browser and you will see your container running.

You should see the following ...
<p align="center">
<img src="images/food.png" width=400px>
</p>

Congratulations, you have now completed Task 1!

5. Before proceeding to the next task clear all of the containers on **node-0** by running the command ```$ docker rm -f $(docker ps -q)```. You should then see that there are no or stopped containers on the host.

```
$ docker rm -f $(docker ps -q)
```



6. Check that there aren't any container running on **host-0**
```
$ docker ps -a
```

## <a name="start-cluster"></a>Task 2: Start a Docker Cluster

Up until this point we have been deploying on a single server. Real-world applications typically run on many hosts. 

Docker has powerful tools to manage multi-host and clustered apps. It has built-in capabilities to define, schedule, and scale services. In the next part of the lab we will use Docker clustering to deploy a multi-service app across several hosts and scale it meet growing demand.

### Set Up Docker Clustering

A Docker Swarm cluster has Swarm managers and Swarm worker nodes. The managers manage and retain the state of the cluster while the worker nodes run application loads. As of Docker 1.12 no external backend or 3rd party components are required for a fully functioning Swarm cluster.

In this part of the demo we will use all three of the nodes in our lab. **node-0** will be our Swarm manager while **node-1** and **node-2** will server as our Swarm worker nodes. Swarm supports a highly available, redundant managers but for the purposes of this lab we will only have a single manager.

### Create a Swarm Master

Get the internal IP address of **node-0**

```
$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 06:67:be:89:1f:b5
          inet addr:172.31.22.238  Bcast:172.31.31.255  Mask:255.255.240.0
...
```

Create a Swarm manager on **node-0** with its internal IP address

```
$ docker swarm init --listen-addr <IP>:4500
 ```

### Join a Worker Node to a Swarm Cluster

Open up a second tab and log in to **node-1** keeping the first tab open

```
$ ssh -i key.pem ubuntu@<node-1 external IP>
fdadfas
```

Join the swarm cluster as a worker node with the internal IP address of the Swarm master, **node-0**

```
$ docker swarm join <node-0 IP>:4500
```

Repeat step 3 for **node-2**

Go back to the **node-0** tab and note that **node-1** has joined the cluster as a worker node

```
docker node ls
ID              NAME                                          STATUS  AVAILABILITY/MEMBERSHIP  MANAGER STATUS  LEADER
1q7l9v5cswpa    ip-192-168-34-90.us-west-2.compute.internal   READY   ACTIVE
2un9gn4ye1uc *  ip-192-168-33-113.us-west-2.compute.internal  READY   ACTIVE                   REACHABLE       Yes
```

```docker node ls``` shows us all of the nodes that are in a given Swarm cluster. Here we can see the hostname, the unique ID of the host, and the role of the host in the cluster. The ```*``` denotes the host that we are currently running the Docker command from.

Repeat the section above on **node-2**

Congratulations, you have now set up a Docker Swarm cluster and completed Task 2!

## <a name="multi-application"></a>Task 3: Deploy a Multi-Service, Multi-Host Application



We will now deploy the same application on a cluster of hosts. We will use the Docker `overlay` networking driver to provide out of the box communication between the hosts.

### Run the Application

1. Create the overlay network for our application.

```
$ docker network create -d overlay ovnet

```

2. Inspect the network we just created with 'docker network inspect ovnet'. We can see the internal subnet that it was automatically given.

3. Deploy a single elasticsearch container as a part of the `es` service.

```
$ docker service create --scale 1 --name es --network ovnet elasticsearch

```

4. Deploy our `foodtruck-web` image from Task 1 using your Dockerhub ID. Run 'docker service create --scale 1 --name web --network ovnet -p 5000:5000 <Your Dockerhub ID>/foodtruck-web`

```
$ docker service create --scale 1 --name web --network ovnet -p 80:5000 markchurch/foodtruck-web
6owy7eqtymnym3cceuqkxbocq
```

5. Now display the services that we have running to show that we have deployed our service correctly.

```
$ docker service ls
ID            NAME  SCALE  IMAGE                     COMMAND
2gom8a7h4qd4  es    1      elasticsearch
6owy7eqtymny  web   1      markchurch/foodtruck-web
```

6. We can drill down in to a specific service with `docker service inspect`

```
$ docker service inspect web

```

7. Now that we have verified that our containers have been succesfully deployed, use the browser to see our application in action. Type `<node-0 IP>:5000` into the address bar of the browser.



### Scale the Application

Your Foodtrucks application is gettin very popular! You will now have to scale it to meet demand. We can scale individual services natively with Docker Swarm.

**Step 1:** Scale the web front end to two web servers with `docker service update`

```
$ docker service update --scale 5 web
```

**Step 2:** Confirm that the additional container have been deployed.

```
$ docker service tasks web
ID                         NAME   SERVICE  IMAGE                     LAST STATE         DESIRED STATE  NODE
d74qivq1q22hycetivps6rfxq  web.1  web      markchurch/foodtruck-web  RUNNING 9 minutes  RUNNING        ip-192-168-33-4.us-west-2.compute.internal
9m4kfwxglx4gd0a18972e1le1  web.2  web      markchurch/foodtruck-web  RUNNING 8 minutes  RUNNING        ip-192-168-33-254.us-west-2.compute.internal
5ww7dqpg99ujjp797dzvh40hl  web.3  web      markchurch/foodtruck-web  RUNNING 9 minutes  RUNNING        ip-192-168-33-4.us-west-2.compute.internal
2ibomlpcs1f3g5jrg4qtuyxm3  web.4  web      markchurch/foodtruck-web  RUNNING 2 seconds  RUNNING        ip-192-168-34-102.us-west-2.compute.internal
06bv6ix5mhmmzolom04npgy5n  web.5  web      markchurch/foodtruck-web  RUNNING 2 seconds  RUNNING        ip-192-168-34-102.us-west-2.compute.internal
```

We can see that all five `web` containers are up and running.

**Step 3:** Now go to your web browser and type in any one of the node's external IP address such as `http://<Any Node External IP>`.

You should be able to see our application live. Hit refresh a couple times. Do you notice that the container ID and internal IP are changing? That is because the Swarm is automtically load balancing on port 5000 for every external IP address in our cluster. 



### Recover From Node Failure
Next we are going to simulate a host failure and we will see how the cluster reacts to it.

**Step 1:** Kill one of the hosts (NIGEL how shoul we best kill or take a node out?)

**Step 2:** Wait for a short period and then run `docker service tasks web` again.


```
$ docker service tasks web
ID                         NAME   SERVICE  IMAGE                     LAST STATE          DESIRED STATE  NODE
d74qivq1q22hycetivps6rfxq  web.1  web      markchurch/foodtruck-web  RUNNING 15 minutes  RUNNING        ip-192-168-33-4.us-west-2.compute.internal
9m4kfwxglx4gd0a18972e1le1  web.2  web      markchurch/foodtruck-web  RUNNING 14 minutes  RUNNING        ip-192-168-33-254.us-west-2.compute.internal
5ww7dqpg99ujjp797dzvh40hl  web.3  web      markchurch/foodtruck-web  RUNNING 15 minutes  RUNNING        ip-192-168-33-4.us-west-2.compute.internal
ezwaylssuo89dvn8nzo518ms8  web.4  web      markchurch/foodtruck-web  RUNNING 52 seconds  RUNNING        ip-192-168-33-254.us-west-2.compute.internal
58fxvts7bwms1r15899pp30wb  web.5  web      markchurch/foodtruck-web  RUNNING 2 seconds   RUNNING        ip-192-168-34-225.us-west-2.compute.internal
```



All of the `web` containers are now running on only two nodes. We can also do a `docker node ls` to see that the killed node is now down.


```
$ docker node ls
ID               NAME                                          MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS  LEADER
0caipoo5flafp *  ip-192-168-34-225.us-west-2.compute.internal  Accepted    Ready   Active        Reachable       Yes
0zvj98g5jc7ne    ip-192-168-34-102.us-west-2.compute.internal  Accepted    Down    Active
11seh6fdqvsj5    ip-192-168-33-4.us-west-2.compute.internal    Accepted    Ready   Active
120azncg278j7    ip-192-168-33-254.us-west-2.compute.internal  Accepted    Ready   Active
```



