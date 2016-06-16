# Windows Server Containers and Docker

> **Difficulty**: Beginner

> **Time**: Approximately 15 minutes

This lab will provide a core introduction to Windows Server Containers and the Docker Engine on Windows. You will pull Docker images for Windows, build an application, Dockerize it, and iterate on it.

You will complete the following steps as part of this lab.

- [Step 1 - Open the project in VS Code](#open_proj)
- [Step 2 - Run the app](#run_app)
- [Step 3 - Create a Dockerfile](#create_dockerfile)
- [Step 4 - Build a Docker image](#build_image)
- [Step 5 - Run a container](#run_container)
- [Step 6 - Modify the app](#modify_app)

# Prerequisites

You will need all of the following to complete this lab:

- Windows Server 2016 Technical Preview 5 with the Containers feature enabled
- Visual Studio Code
- Docker Engine
- An RDP client installed on your local laptop

All of these should be installed and ready to use on your Azure VM.

# <a name="open_proj"></a>Step 1: Open the project in VS Code

In this step you'll open the sample app in Visual Studio Code (VS Code).

1. If you are not already logged on to the Azure VM for the Windows Server Container lab, do so now. You can find the connection information in the registration email you received. 

2. Open VS Code by double-clicking the **Visual Studio Code** icon on the desktop.

3. Click **File** > **Open Folder...**.

4. Navigate to `C:\` and **click once** on the `My_App` folder.

5. Click **Select Folder**.

  This will open the contents of the entire folder (the *project*) in VS Code.

6. Click the **Program.cs** file in the **EXPLORER** pane on the left side of VS Code.

  This will load the sample app into the editor window on the right. You will see that it is a simple "Hello World" application written in C#.

  > **Note:** If the **TaylorsConsoleWriter** line of code is not commented out, comment it out by adding `//` (two forward slashes) to the start of the line. Then press `Ctrl+S` to save the change.

# <a name="run_app"></a>Step 2: Run the app

In this step you will run the app to make sure it works.

1. Open a command prompt and run the following command

  ```
  > dotnet run -p C:\My_App\ConsoleApp

  Project ConsoleApp (.NETCoreApp,Version=v1.0) will be compiled because inputs were modified

  Compiling ConsoleApp for .NETCoreApp,Version=v1.0

  Compilation succeeded.
    0 Warning(s)
    0 Error(s)

  Time elapsed 00:00:08.9393947

  Hello World!
  ```
    Noteice the **Hello World!** at the end of the output.

Congratulations. The application works and is ready to be Docerized.

# <a name="create_Dockerfile"></a>Step 3: Create a Dockerfile

In the next two steps you will Dockerize the app. The first step in doing this is to create a text file called **"Dockerfile"** that contains all of the instructions required to create a Docker image containing the app.

The name of the file - **Dockerfile** - is case-sensitive, so be sure to use a capital "D" when saving the file.

1. In VS Code click **File** > **New File**

2. Go to the bottom right of the editor window and click where it says **Plain Text**.

3. Type **Dockerfile** into the text bar that appears at the top of the screen and press **Enter**.

  This will turn on language support features for Dockerfile.

4. Add the following lines to your Dockerfile.

  ```
  FROM microsoft/dotnet
  ADD ConsoleApp/ConsoleApp
  CMD dotnet run -p C:\Consoleapp
  ```

5. Save the file as `C:\My_App\Dockerfile`. Remember to use a capital **D** at the beginning of the filename.

# <a name="build_image"></a>Step 4: Build a Docker image

Now that you have your **Dockerfile** ready, the second step in Dockerizing your app is to create a Docker image. In this step you will use the Dockerfile created in the previous step to create a Docker image containing the application.

1. If you haven't already done so, open a command prompt and change into the `C:\My_App` directory.

2. Type the following `docker build` command.

  ```
  > docker build -t app:1 C:\My_App

  Sending build context to Docker daemon 8.056 MB
  Step 1 : FROM microsoft/dotnet
  ---> d426874c21c8
  Step 2 : ADD ConsoleApp /ConsoleApp
  ---> 3f4dacc9794e
  Removing intermediate container 45aa6f66fcb5
  Step 3 : CMD dotnet run -p C:\Consoleapp
  ---> Running in 6c853c272f08
  ---> c7fb54d4b8f6
  Removing intermediate container 6c853c272f08
  Successfully built c7fb54d4b8f6
  ```

    This command sends the contents of `C:\My_App` to the Docker daemon as the build context and builds an image called **app** tagged with **1** (`app:1`).

3. Run a `docker images` command to verify the presence of the tagged image.

  ```
  > docker images
  REPOSITORY   TAG    IMAGE ID        CREATED           SIZE
  app          1      972fba16c34f    10 seconds ago    10.16 GB
  ```

Well done. You have successfully Dockerized the Hello World app.

# <a name="run_container"></a>Step 5: Run a container

In this step you will take the image created in the previous step and launch a new container from it.

1. Run the following `docker run` command.

  ```
  > docker run --rm app:1
  ```
  The `--rm` flag tell the Docker daemon to remove the container once it has executed the application (printing "Hello World!" to the terminal). The `app:1` tells it to base the container on the image created in the previous step.

  You will see the following text printed to the terminal.

  ```
  Project Consoleapp (.NETCoreApp,Version=v1.0) will be compiled because Input items removed from last build
  Compiling Consoleapp for .NETCoreApp,Version=v1.0

  Compilation succeeded.
    0 Warning(s)
    0 Error(s)

  Time elapsed 00:00:13.2652112

  Hello World!
  ```

Congratulations. You have launched a Windows Container based off the Dockerized application you created in the previous steps.

# <a name="modify_app"></a>Step 6: Modify the app

In this step you'll modify the app, rebuild it, Dockerize it again, and test it.

1. In VS Code, expand the **ConsoleApp** folder and open the **Program.cs** file.

2. Uncomment the **TaylorsConsoleWriter** line by removing the two forward slashes `//` at the beginning of the line.

3. Save your changes.

4. Rebuild the app with the following command.

  ```
  > dotnet restore C:\My_App\ConsoleApp
  ```

5. Create a new Docker image using the same **Dockerfile** as last time.

  ```
  docker build --no-cache -t app:2 C:\My_App
  ```

6. Run a `docker images` to verify the previous step.

  ```
  > docker images

  REPOSITORY   TAG    IMAGE ID         CREATED             SIZE
  app          2      326c7a79c883     13 seconds ago      10.16 GB
  app          1      972fba16c34f     9 minutes ago       10.16 GB
  ```

    As you can see, you now have two tagged images in the local **app** repo.  `app:2` is the image you just created and will have the updated contents of `C:\My_App`.

7. Start a new container from the `app:2` image.

  ```
  > docker run --rm app:2

  Project Consoleapp (.NETCoreApp,Version=v1.0) will be compiled because Input items removed from last build
  Compiling Consoleapp for .NETCoreApp,Version=v1.0

  <SNIP>

  Hello World!
  Unhandled Exception: System.DllNotFoundException: Unable to load DLL 'msvcr120.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)
   at TaylorsConsoleWriter._snwprintf(StringBuilder str, IntPtr length, String format)
   at TaylorsConsoleWriter.WriteLine(String StringToOutPut)
   at ConsoleApplication.Program.Main(String[] args)
  ```

  Uh oh! An error.

  The changes you made to the application (adding the **TaylorsConsoleWriter** line) require the **msvcr120.dll** file. However, the file is not present inside the image we created. This is because the DLL is included with Visual Studio, so it's not obvious that it needs explicitly adding to the project.

8. Add the dependency to the image by adding the following two lines to your Dockerfile immediately below the **FROM** instruction.

  ```
  ADD vcredist_x64.exe /vcredist_x64.exe
  RUN start /wait c:\vcredist_x64.exe /q /norestart
  ```

  Adding these lines will add the **vcredist_x64.exe** file into the image and then execute it. This will install the missing DDL to the image.

  Your **Dockerfile** should now look like this.

  ```
  FROM microsoft/dotnet
  ADD vcredist_x64.exe /vcredist_x64.exe
  RUN start /wait c:\vcredist_x64.exe /q /norestart
  ADD ConsoleApp /ConsoleApp
  CMD dotnet run -p C:\Consoleapp
  ```

9. Save the changes to your Dockerfile.

10. Create an updated image tagged as **3** using the updated Dockerfile.

  ```
  > docker build --no-cache -t app:3 C:\My_App
  ```

11. Start a container form the new tagged image.

  ```
  > docker run --rm app:3
  ```

Congratulations. You've fixed the missing dependency and your Dockerized app is working again.

# Additional Resources

You can refer to the following resources for more information and help:
- Documentation: http://aka.ms/containers
- Docker: http://www.docker.com
- Forum: https://aka.ms/containers/forum
- Videos: https://aka.ms/containers/videos
