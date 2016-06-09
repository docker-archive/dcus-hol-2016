# Task 4 - Deploy a simple application on UCP

## Pre-requisites
- You must have [Docker Toolbox](https://www.docker.com/products/docker-toolbox) installed on your local machine

## Deploy an application using Docker Compose

In this exercise we will deploy a simple multi container application. The application will be run using **Docker Compose** and contains 2 services (containers)
- A Redis Container
- A Java client that pings the container to get a response


1. SSH into your UCP controller AWS machine
2. Use Git to clone the application repository from https://github.com/johnny-tu/HelloRedis.git 

   ```bash
   ubuntu@ucp-controller:~$ git clone https://github.com/johnny-tu/HelloRedis.git
   Cloning into 'HelloRedis'...
   remote: Counting objects: 45, done.
   remote: Total 45 (delta 0), reused 0 (delta 0), pack-reused 45
   Unpacking objects: 100% (45/45), done.
   Checking connectivity... done.

   ```
3. You should now see a folder called `HelloRedis`. Change directory into this folder
4. List the files in the directory. You should see a `docker-compose.yml` file

   ```bash
   ubuntu@ucp-controller:~/HelloRedis$ ls
   docker-compose.prod.yml  docker-compose.yml  Dockerfile  lib  README.md  src

   ```
   
5. Run `docker-compose up -d`

   ```bash
   ubuntu@ucp-controller:~/HelloRedis$ docker-compose up -d
   Pulling redis (redis:latest)...
   latest: Pulling from library/redis
   ...
   ...
   Digest: sha256:ddb31adb1f61405ee59fc9c569c2b09f1796d0f2190e78d84b873f2d930236c1
   Status: Downloaded newer image for redis:latest
   Creating helloredis_redis_1
   Building javaclient
   Step 1 : FROM java:7
    ---> 9f4357ff2eef
   Step 2 : COPY /src /HelloRedis/src
    ---> eabdb34fe077
   Removing intermediate container fd05659c730a
   Step 3 : COPY /lib /HelloRedis/lib
    ---> 16a65528b2ce
   Removing intermediate container 9f52240204e6
   Step 4 : WORKDIR /HelloRedis
    ---> Running in 686c4bd65d85
    ---> 01d9a0c2a101
   Removing intermediate container 686c4bd65d85
   Step 5 : RUN javac -cp lib/jedis-2.1.0-sources.jar -d . src/HelloRedis.java
    ---> Running in f829db30c628
    ---> 9f08dd8a743c
   Removing intermediate container f829db30c628
   Step 6 : CMD java HelloRedis
    ---> Running in 0e070b1cd994
    ---> 9dc249ef701d
   Removing intermediate container 0e070b1cd994
   Successfully built 9dc249ef701d
   Creating helloredis_javaclient_1

   ```
   
6. Now run `docker-compose ps`. What can you observe?

   ```bash
   ubuntu@ucp-controller:~/HelloRedis$ docker-compose ps
            Name                        Command               State    Ports
   ---------------------------------------------------------------------------
   helloredis_javaclient_1   java HelloRedis                  Up
   helloredis_redis_1        docker-entrypoint.sh redis ...   Up      6379/tcp
   ```

7. Switch over to UCP on your web browser
8. Click on the “Applications” link on the left navigation bar
9. You should see the following output

   ![](images/ucp02_t4_applications.PNG)

10. Click on the “Show Containers” link on the right side to expand the view of the application

   ![](images/ucp02_t4_applications_expanded.PNG)
   
   You will notice that when we expand the the view of the `helloredis` application, we also see each service that the application is composed of. 

## Using the Client Bundle

Manually logging into the EC2 instance to run docker-compose to deploy your applications is not very convenient and in a lot of cases not possible. 
After all, you wouldn’t want to give SSH access to too many people. So instead of SSHing into the machine in order to deploy our applications, 
we use the client bundle.

The client bundle sets up the certificates needed in order to allow us to use a Docker client or docker-compose on our local machine. 
It will connect our Docker client to the Swarm manager that is running on our UCP controller node. 

1. Navigate to your user profile in UCP

   ![](images/ucp02_t4_profile_dropdown.PNG)
2. Click the “Create a Client Bundle” button. This will download a “zip” file with the necessary keys, 
   certificates and scripts needed to connect your Docker client to Swarm. 
   
   ![](images/ucp02_t4_client_bundle.PNG)
3. Unzip the Client Bundle to a folder of your choice
4. Take note of the files in your folder now. You should see an `env.sh` file

   Note that in this example we are using a Windows machine and we have unzipped the bundle to `C:\Docker\ucp_client_bundles\ucp-bundle-admin`. 
   Let's examine the directory using the Windows Terminal (CMD)
   
   ```
   C:\Docker\ucp_client_bundles\ucp-bundle-admin>dir
    Volume in drive C is OS
    Volume Serial Number is 10D0-EAA3

    Directory of C:\Docker\ucp_client_bundles\ucp-bundle-admin

   20/05/2016  04:54 PM    <DIR>          .
   20/05/2016  04:54 PM    <DIR>          ..
   20/05/2016  04:52 PM             3,684 ca.pem
   20/05/2016  04:52 PM             5,392 cert.pem
   20/05/2016  04:52 PM               450 cert.pub
   20/05/2016  04:52 PM               623 env.cmd
   20/05/2016  04:52 PM               662 env.ps1
   20/05/2016  04:52 PM               609 env.sh
   20/05/2016  05:23 PM    <DIR>          HelloRedis
   20/05/2016  04:52 PM             1,679 key.pem
               7 File(s)         13,099 bytes
               3 Dir(s)  595,220,852,736 bytes free
			  
   ```
   
   Note the certificates and the `env.sh` and `env.cmd` files. For users on Mac OSX or Linux, you will be using the `env.sh` file. Windows users using
   `CMD` terminal will be using `env.cmd`

5. Open `env.sh` and take note of the environment variables that the script is setting
   Let's take a look at `env.sh`
   ```
   export DOCKER_TLS_VERIFY=1
   export DOCKER_CERT_PATH="$(pwd)"
   export DOCKER_HOST=tcp://ec2-54-187-21-127.us-west-2.compute.amazonaws.com:443
   ```
   
   The `DOCKER_TLS_VERIFY` variable turns on TLS verification between our Docker Client and the Docker Engine we want to communicate with.  
   The `DOCKER_CERT_PATH` variable specifies where our SSL certificates and private key is located. In this case it is in the same folder as our `env.sh` script.  
   The `DOCKER_HOST` variable specifies the address of the Host we are connecting the client to. In this case, it is our Swarm Manager running on the UCP
   controller node. 
   
   Note that in our example the `DOCKER_HOST` is specified with the public IP address of the node. It would also be possible to specify it with the DNS Name
6. Open your terminal and change directory into the folder where you extracted the client bundle zip
7. Run `source ./env.sh` or `env.cmd`

   **For Mac and Linux users**  
   `$ source ./env.sh`  
   To verify that it worked, check the value of the variables
   ```
   $ echo $DOCKER_HOST
   tcp://ec2-54-187-21-127.us-west-2.compute.amazonaws.com:443
   ```
   
   **For Windows users using CMD Terminal**  
   ```
   C:\Docker\ucp_client_bundles\ucp-bundle-admin>env.cmd

   C:\Docker\ucp_client_bundles\ucp-bundle-admin>echo %DOCKER_HOST%
   tcp://ec2-54-187-21-127.us-west-2.compute.amazonaws.com:443
   ```
   **Note:** Windows users can opt to use a command line tool such as `git bash` and thus be able to follow the same instructions as Mac and Linux users.
8. Now run `docker info`. You should be able to see all nodes that are connected   

   ```
   C:\Docker\ucp_client_bundles\ucp-bundle-admin>docker info
   Containers: 17
    Running: 17
    Paused: 0
    Stopped: 0
   Images: 41
   Server Version: swarm/1.1.3
   Role: primary
   Strategy: spread
   Filters: health, port, dependency, affinity, constraint
   Nodes: 3
    ucp-controller: 10.0.15.6:12376
      Status: Healthy
      Containers: 12
      Reserved CPUs: 0 / 1
      Reserved Memory: 0 B / 3.859 GiB
      Labels: executiondriver=, kernelversion=4.2.0-23-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
      Error: (none)
      UpdatedAt: 2016-05-24T05:04:06Z
    ucp-node-0: 10.0.7.110:12376
      Status: Healthy
      Containers: 2
      Reserved CPUs: 0 / 1
      Reserved Memory: 0 B / 3.859 GiB
      Labels: executiondriver=, kernelversion=4.2.0-23-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
      Error: (none)
      UpdatedAt: 2016-05-24T05:04:20Z
    ucp-node-1: 10.0.28.145:12376
      Status: Healthy
      Containers: 3
      Reserved CPUs: 0 / 1
      Reserved Memory: 0 B / 3.859 GiB
      Labels: executiondriver=, kernelversion=4.2.0-23-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
      Error: (none)
      UpdatedAt: 2016-05-24T05:04:23Z
   Cluster Managers: 1
    10.0.15.6: Healthy
      Orca Controller: https://10.0.15.6:443
      Swarm Manager: tcp://10.0.15.6:2376
      KV: etcd://10.0.15.6:12379
   Plugins:
    Volume:
    Network:
   Kernel Version: 4.2.0-23-generic
   Operating System: linux
   Architecture: amd64
   CPUs: 3
   Total Memory: 11.58 GiB
   Name: ucp-controller-ucp-controller
   ID: 44WM:6P6I:N6WZ:U4OB:RFMN:WKY7:OX3N:3OSU:GTE3:3SDC:FROD:OQ6Z
   Labels:
    com.docker.ucp.license_key=4dd1umru9iT8lplNkXi5I1cdZrdxBIl60qyNzB9i6x_b
    com.docker.ucp.license_max_engines=10
    com.docker.ucp.license_expires=2016-05-31 21:53:37 +0000 UTC
   ```
9. Clone the HelloRedis repository https://github.com/johnny-tu/HelloRedis.git into another folder of your choice
9. Remove the existing HelloRedis application from UCP.

   ![](images/ucp02_t4_applications_remove.PNG)
   
10. Launch the application by using the Client Bundle. To do this, you just need to go into the applications folder and run `docker-compose up -d`

   You may notice the following error
   ```
   johnny@JT MINGW64 ~/Documents/GitHub/HelloRedis (master)
   $ docker-compose up -d
   helloredis_redis_1 is up-to-date
   Creating helloredis_javaclient_1
   unable to find a node that satisfies image==helloredis_javaclient
   ```  
   If you look inside the `docker-compose.yml` file, you will notice that the `javaclient` service is defined with a `build` instruction. This is not advised on productions runs, especially when 
   Docker Compose is interacting with Swarm or UCP. Compose does not have the ability to build an image across every node in the Swarm cluster. It will build on the node the container is scheduled on.
   
   Sometimes this can lead to errors if Swarm tries to schedule a container on a node without the image. 

   For best practice, in a production deployment of an application, the compose file should only use images already built and available through Docker Hub or DTR

   Take a look inside the `docker-compose.prod.yml` and compare the difference.

   Now run, `$docker-compose -f docker-compose.prod.yml up -d` to launch the application. 

   The `-f` option allows users to specify a specific compose file to use   

11. Take note of the nodes each container is running on. You should see that both containers are on the same node. This is the expected behavior due 
   to how networking works on older Compose applications. Later in the course we will learn how to deploy application containers across multiple nodes.

   ![](images/ucp02_t4_helloredis.PNG)
 
## Deploy another application   

For the following section, use what you have learnt just now and complete the steps listed below.
   
1. Now that you’ve deployed your first application, it’s time to try another example. Go to https://github.com/prakhar1989/FoodTrucks
2. Clone the `FoodTrucks` repo into your local PC or Mac
3. Deploy the `FoodTrucks` application into UCP. Remember to use the Client Bundle
4. View the application in your web browser

   If you completed all the steps correctly, you should see a very cool application that allows you to search for food trucks in San Francisco
   
   ![](images/ucp02_t4_foodtrucks.PNG)
   




   

