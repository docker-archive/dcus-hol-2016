#Orchestration with Docker Clustering

#####Prerequisites
* You will be using __node0__, __node1__, and __node2__
* Ensure that no containers are running on these nodes ```$ docker rm -f $(docker ps -q)```
* 



##Task 1: Deploy a Simple Application
#####Set Up Environment

1. Connect to __host0__
2. Use the following command to clone the simpleApp repo from GitHub to node-0

```
node-0:$ git clone https://github.com/mark-church/FoodTrucks.git
Cloning into 'dockchat'...
remote: Counting objects: 54, done.
remote: Total 54 (delta 0), reused 0 (delta 0), pack-reused 54
Unpacking objects: 100% (54/54), done.
Checking connectivity... done.
```
3. Change directory to ```FoodTrucks``` and examine the list of files in the repo

```
$ cd simple
$ tree
```
SimpleApp is a basic Flask application. It has a built in webserver that serves up a webpage at the address that the application is bound to. It's comprised of several python files in addition to _index.html_ which is the template file for page that is served by the application. 

The other file in this directory is the __Dockerfile__. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. This includes packages that the application will need, directories of data that needs to be a part of the image, and any metadata that should be shipped in the image. The Docker engine uses Dockerfiles to create new container images.

Next we will use a Dockerfile to create an image and then run a container from that image.

#####Build the Application
1. Verify that the Docker is running on __host0__

```
$ docker version
Client:
 Version:      1.11.1
 API version:  1.23
 Go version:   go1.5.4
 Git commit:   5604cbe
 Built:        Wed Apr 27 00:34:20 2016
 OS/Arch:      darwin/amd64

Server:
 Version:      1.11.1
 API version:  1.23
 Go version:   go1.5.4
 Git commit:   8b63c77
 Built:        Tue May 10 10:39:20 2016
 OS/Arch:      linux/amd64
```


2. Inspect the Dockerfile to see how its parameters will build the container image

```
$ cat Dockerfile
FROM ubuntu

RUN sudo apt-get update && apt-get -y install python-pip

RUN sudo pip install flask

COPY / /simpleDir

WORKDIR /simpleDir

RUN chmod a+x /simpleDir/run.py

CMD ["python", "/simpleDir/run.py"]
```
__FROM__:

__ADD__:

__RUN__:

__CMD__:



2. Build the image specifying the directory of the Dockerfile and tagging the image with a name

```
$ docker build -t simple-app .
ending build context to Docker daemon 46.41 MB
Step 1 : FROM ubuntu
 ---> 97434d46f197
 ...
 
 done!
 ```
 
3. See that the image is now in the local image repository of the Docker engine

```
$ docker images

```


#####Run the Application
1. Now create a container with our simple-app image and run it on __host0__ at port 80

```
$ docker run -it -p 80:80 simple-app
```

2. Verify through the Docker CLI that the container is running

```
$ docker ps


```
From this output we can see the following:   

* The container has been given an automatically generated ID and name (we can specify the name if we choose)
* We can see what image was used to create the container
* We can also see if and how the container is exposed outside the host (in this case on port 80 of the host interfaces)

3. Lastly, let's use the browser to connect to our live container. Look up the public facing IP address of __host0__. Type ```http://<ip address>:80''' into the browser and you will see your container running.




##Task 2: Start a Docker Cluster
Up until this point we have been dealing with a singel-container application on one host. Real-world applications are typically many apps across hosts that may even be in different environments. The Docker ecosystem has powerful tools to scale and manage large applicaion


#####Set Up Docker Clustering
1. Create cluster
2. Point client at cluster address


#####Swarm Cluster Architecture

1. ```docker version``
2. ```docker info```
3. ```docker network ls```


##Task 3: Deploy a Multi-Service, Multi-Host Application

#####Deploy Application
1. Examine compose file

```
$ cat compose.yml

```
######Compose Deployment Options
__image__:

__build__:

__expose__:

__ports__:

__networks__:

__volumes__:


1. ```docker login```
2. ```docker-compose build build-compose.yml```
3. ```docker push```
4. ```docker-compose run-compose.yml up```
5. ```docker ps``` 
6. Go to application

#####Explore Application
1. ```docker inspect container```
2. ```docker network inspect net```
3. show labels on worker nodes and infrastructure node

#####Cluster and Application Operations
1. Scale front-end web server to 3 
2. Hit front-end and show different containers serving up the web page
3. Rolling update of application
4. Kill host
5. ```docker ps``` and see container rescheduled





