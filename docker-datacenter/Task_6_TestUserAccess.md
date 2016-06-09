# Task 2 - Test User Access

In this task you will complete the following steps:
1. Test permission labels
2. Test container access from the web UI
3. Test container access from the command line
4. Test admin access form the command line
5. Test default permissions

## Pre-requisites

- Completed Task 1

## Step 1 - Test permission labels

Docker UCP uses labels to implement permissions and access control. In the previous lab (Task 1) you deployed the "nginx1" container with the "view" label. You also assigned the Engineering team "View Only" access to all resources tagged with the "view" label. In this step you will log back in to UCP as "johnfull" and verify that you only have view access to the "nginx1" container.

1. Logout of UCP and log back in as user **johnfull**

2. Click on the **Containers** link in the left pane.

  Confirm that you can only see the "nginx1" container (with the "view" label). The other containers that you deployed with no labels will not be visible.

3. Click the controls button to the right of the container (three dots) and attempt to **Stop** the container. The action will fail and you will see an error message like the one shown below.

  ![](http://i.imgur.com/a8moPah.png)

4. Click on the container to view its details.

5. Scroll down to the **Labels** section and verify the presence of the **view** label.

  ![](http://i.imgur.com/R5S8B3a.png)

6. Click the **Containers** link in the left pane.

7. Click the **+ Deploy Container** button to deploy a new container with the following basic options.

  ![](http://i.imgur.com/tgUgqQq.png)

  When you click the **Run Container** button, the deployment will fail. This is because members of the Engineering team only have *View Only* access to resources with the **view** label. They cannot create containers with the **view** label.

  ![](http://i.imgur.com/Bn4gI5T.png)

8. Repeat the previous action two more times, but configure the containers as shown in the table below:

| Image Name | Container Name  | Permissions Label |
| :--------- | :---------------| :---------------- |
| ubuntu     | ub1             | restricted        |
| ubuntu     | ub2             | run               |

  Deploying with either of these two labels will work. This is because members of the Engineering team have *Restricted Control* on the **restricted** label, and *Full Control* on the **run** label. Both of these permissions allow for the deployment of new containers.


## Step 2 - Test container access from the web UI

In this step you will attempt to perform certain actions while logged in as the **johnfull** user. Depending on which permissions labels are in force will determine whether these actions succeed or fail.

1. Click on the container **ub1**. Then click the **Console** tab.

  ![](http://i.imgur.com/D2eedEJ.png)

2. Click on the **Run** button with "bash" specified in the field.

  This action is the GUI equivalent of running a `docker exec` command. In this case, you are trying to execute a `bash` terminal inside the **ub1** container.

  You will get an error message saying *Error attempting to open exec session*. This is because the you are logged in as **johnfull** who is a member of the **Engineering** team, and the **Engineering** team only have *Restricted Control* to the **ub1** container via the **restricted** label. *Restricted Control* does not allow you to open exec sessions to a container.

4. Now try the same thing with the **ub2** container.

   This time the bash terminal will launch successfully. This is because the user **johnfull** is a member of the **Engineering** team which has *Full Control* over the **ub2** container via the **run** label.

  ![](http://i.imgur.com/IPQYt6Z.png)

## Step 3 - Test container access from the command line

In this step you will create and download a **client bundle** for the **johnfull** user, connect to UCP using the client bundle, and perform some tests from the command line. You can do this from either a Windows or Mac. The steps below are from a Windows machine.

1. Click the **johnfull** dropdown in the top right corner and select **Profile**.

2. Scroll to the bottom of the profile screen and click **Create a Client Bundle**.

  This will download the client bundle to your local machine as a zipped archive file.

3. Unzip the contents of the archive file and open a command prompt to the location of the extracted contents. On a Windows machine this is likely to be C:\Users\your-user\Downloads\ucp-bundle-johnfull.

  The examples in this tutorial are using Git Bash. Feel free to use a command line tool of your choice.

4. List the files in your current directory.

   ```
   nigel@surfacewah MINGW64 ~/Downloads/ucp-bundle-johnfull
   $ ls
   ca.pem  cert.pem  cert.pub  env.cmd  env.ps1  env.sh  key.pem
   ```

5. Execute the `source.sh` shell script.

  ```
  nigel@surfacewah MINGW64 ~/Downloads/ucp-bundle-johnfull
  $ source env.sh
  ```

6. Run a `docker ps` command to list the containers.

  The output should contain the "nginx1", "ub1", and "ub2" containers created in the previous steps.

  ```
  nigel@surfacewah MINGW64 ~/Downloads/ucp-bundle-johnful
  $ docker ps
  CONTAINER ID   IMAGE      COMMAND                  CREATED             STATUS              PORTS             NAMES
  26483416a3ef   ubuntu     "/bin/bash"              About an hour ago   Up About an hour                      ub2
  656e3e4c0ccf   ubuntu     "/bin/bash"              About an hour ago   Up About an hour                      ub1
  0fe093853832   nginx      "nginx -g 'daemon off"   About an hour ago   Up About an hour    80/tcp, 443/tcp   nginx1
  ```

7. Run `docker exec -it ub1 bash` to open a `bash` terminal on the "ub2" container.

   This will result in an *Error response from daemon: access denied* error. This is because the "ub1" container is tagged with the "restricted" label, which maps the *Restricted Control* permission to members of the Engineering team.

8. Repeat the same command for the **ub2** container.

   This time the command works because the **ub2** container is tagged with the **run** label which maps the *Full Control* permission to members of the Engineering team. Restricted Control does not allow users to `docker exec` into a container.


## Step 4 - Test admin access form the command line

In this step you will attempt to launch a console form the UCP web UI, as well as the **admin** user's client bundle.

1. Logout of UCP as **johnfull** and log back in as the **admin** user

2. Click the **Containers** link in the left of the web UI and click the **ub1** container.

3. Click the **Console** tab and then click the **Run** button to run a `bash` terminal.

   This time the terminal opens. This is because the "admin" user has full access to all UCP resources, regardless of permissions labels that are applied.

4. Download the Admin user's client bundle and unzip to a folder of your choice.  See steps 3.1-3.3 for details of how to do this.

5. Open a command prompt to the location of the unzipped client bundle and execute the `env.sh` script.

6. Run a `docker ps` command and take note of how many containers you can see.

   You should see the **nginx1**, **ub1** and **ub2** containers that were launched in Step 1. You should also see any additional containers that you launched without permissions labels at the end of Task 1.

7. Run `docker exec` and open a `bash` terminal to the "ub1" container

   The operation will also succeed because you are connected to UCP as the **admin** user.

## Step 5 - Test default permissions

In this step you will test access to UCP resources that are not tagged with permissions labels. The actions in this step wil be performed in the Docker UCP web UI.

1. Logout of UCP as the **admin** user and log back in as **johnfull**.

2. Click on the **Images** link and click **Pull Image**.

3. Pull the "hello-world" image.

  ![](http://i.imgur.com/NjJgEfH.png)

  The image pull operation will be successful.

4. Click on the **Networks** link and click **+ Create Network** to create a new network called "johns-net".

  Just give the network a name and click **Create**.

  The network will be successfully created.

From the previous 4 steps we can see that the user **johnfull** has full access to create networks, pull images, and perform other UCP tasks. This is because **johnfull** has the *Full Access* default permission, giving him full access to all non-tagged UCP resources. His access is only restricted to resources tagged with permissions labels.

5. Logout of UCP as **johnfull** and log back in as **kerryres**.

6. Click on the **Images** link and pull the "alpine" image.

7. Click on the **Networks** link and create a network called "kerry-net".

  Similar  to **johnfull**, **kerryres** can also pull images and create networks despite only having the **Restricted Control** default permission. However, there are actions that users with Full Control can do, that users with Restricted Control cannot do such as `docker exec` into containers and lauch **privileged** containers.

8. Logout of UCP as **kerryres** and log back in as **barryview**.

9. Click on the **Images** link.

  Notice that Barry does not even have a **Pull Image** button. This is because **barryview** has the **View Only** default permission. This permission does not allow operations such as pulling images.

10. Click the **Networks** link and create a network called "barry-net".

   You will get an **Error creating network: access denied** error message because of insufficient permissions.

11. Logout of UCP as **barryview** and login as **traceyno**.

12. Notice that Tracey only has links to the following three resource types:
   - Applications
   - Containers
   - Nodes

    This is because Tracey has the **No Access** default permission. However, because Tracey is a members of the Engineering team, she gets access to all of the tagged resources that the Engineering team has access to.

13. Click the **Containers** link and notice that Tracey can see the three containers that have the **view** label attached to them.
  
  


   
