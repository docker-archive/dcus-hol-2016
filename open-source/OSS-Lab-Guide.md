#Orchestration with Docker Clustering
##Task 1: Deploy a Simple Application
#####Set Up Environment
1. Use the following command to clone the DockChat repo from GitHub to node-0

```
node-0:$ git clone https://github.com/mark-church/simple-site.git
Cloning into 'dockchat'...
remote: Counting objects: 54, done.
remote: Total 54 (delta 0), reused 0 (delta 0), pack-reused 54
Unpacking objects: 100% (54/54), done.
Checking connectivity... done.
```
2. Change directory to ```simple-site``` and examine the list of files in the repo

```
$ tree
```
#####Build the Application
1. ```docker version```
2. Examine Dockerfile

```
cat Dockerfile
```
__FROM__:

__ADD__:

__RUN__:

__CMD__:



2. Build the image ```docker build```
3. ```docker images```

#####Run the Application
1. ```docker run simple-site```
2. ```docker ps```
3. ```docker port simple-site```



3. 

##Deploy a Multi-Service, Multi-Host Application

#####Set Up Docker Clustering
1. Create cluster
2. Point client at cluster address


#####Swarm Cluster Architecture

1. ```docker version``
2. ```docker info```
3. ```docker network ls```

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


1.```docker login```
2. ```docker-compose build build-compose.yml```
3. ```docker push```
4. ```docker-compose run-compose.yml up```
5. ```docker ps``` 
6. Go to application

####Explore Application
1. ```docker inspect container```
2. ```docker network inspect net```
3. show labels on worker nodes and infrastructure node

#####Infrastructure Operations
1. Scale front-end
2. Hit front-end and show different containers serving up the web page
3. Rolling update of application
4. Kill host
5. ```docker ps``` and see container rescheduled





