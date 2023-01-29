===========================================================================

**CONTENTS**

[1. Lab Setup](#1)

[2. Installing Docker Engine on Linux machine](#2)

[3. Cloning a repository](#3)

[4. Docker images ](#4)

[5. Docker containers ](#5)

[6. Dockerfile ](#6)

[7. Dockercompose ](#7)

===========================================================================

**INTRODUCTION**

Container is roughly analogous to the Virtual Machine, and like VMs, containers live on top of a host machine and use its resources, however, instead of virtualising the underlying hardware, they virtualise the host OS. Meaning containers don’t need to have their own OS, making them much more lightweight than VMs, and consequently quicker to spin up. 

Docker is the containerization platform that is used to package your application and all its dependencies together in the form of containers to make sure that your application works seamlessly in any environment which can be developed or tested or in production. Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. 

===========================================================================

**DESCRIPTION OF TASKS**

This is intended to help get your feet wet with the basics of Docker by presenting a number of tasks.

Basically, the tasks for you is to build Docker containers:

**1 database container database**

 - database image: microsoft sqlserver 2019 ubuntu based

 - sa password: zaQ@123456! 

 - version: express

 - port:
      - container: 1433
      - published: 1434

 - install node exporter and will automatically run when starting the container

 - must have a volume to store data for backup.

**1 web api container that connects to the database**

 - Web api needs to configure connection string to connect to database:

   ConnectionStrings__AppDatabase=Server={sqlserverIp};Database=AppDatabase;User Id={user};Password={Password};
   
  - port:
      - container: 80
      - published: 10005

**1 web app container**

 - Web app UI needs to display the web api 
 
   WebApi={ApiUrl}

 - port:
     - container: 80
     - published: 10000

First practice using docker command normally to build images and run the containers, then using docker compose to deploy all the apps together.

===========================================================================
## 1. Lab Setup <a name="1"></a>

Create Azure Linux VM with OS image of **Ubuntu Server 20.04 LTS**. I am going to name this VM **ub01** You can choose the name of VM as you want. 

![lab1-1](https://raw.githubusercontent.com/vottri/Docker/main/images/lab1-1.png)

Allow your VM's port **22** (SSH) to be accessible from the Internet.

![lab2](https://raw.githubusercontent.com/vottri/Docker/main/images/lab2.png)

Use **PuTTY** for connecting to the Linux Virtual Machine. Enter your Linux machine's public IP address in Host Name and Port will be 22. Click on **Open** button to connect.

![lab3](https://raw.githubusercontent.com/vottri/Docker/main/images/lab3.png)

When you are inside the Linux VM:

![lab3-1](https://raw.githubusercontent.com/vottri/Docker/main/images/lab3-1.png)

Check for working Internet connection.

Update your system. 

```sh
sudo apt-get update
```

Install some required packages:

```sh
sudo apt-get install -y wget apt-transport-https
```

## 2. Installing Docker Engine on Linux machine <a name="2"></a>

Docker provides a convenience script at https://get.docker.com/ to install Docker. By default, the script installs the latest stable release of Docker, containerd, and runc.

Downloads the script from https://get.docker.com/ and runs it to install the latest stable release of Docker on your Linux machine:

```sh
wget -qO- https://get.docker.com/ | sh
```

![d1](https://raw.githubusercontent.com/vottri/Docker/main/images/d1.png)
![d2](https://raw.githubusercontent.com/vottri/Docker/main/images/d2.png)

To run Docker as a non-root user, create a Unix group called **docker** and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the **docker** group. On some Linux distributions, the system automatically creates this group when installing Docker Engine using a package manager. In that case, there is no need for you to manually create the group.

Add your user to the **docker** group.

```sh
sudo usermod -aG docker cloud_user
```

Activate the changes.

```sh
newgrp docker
```

Try to run docker commands without sudo.

```sh
docker run hello-world
```
![d3](https://raw.githubusercontent.com/vottri/Docker/main/images/d3.png)

## 3. Cloning a repository <a name="3"></a>

### Create a Personal Access Token

Visit GitHub and sign into your account. Click on your user icon in the upper-right hand corner and select **Settings** from the drop down menu.

On the page that follows, locate the **Developer settings** section on the left-hand menu and click **Personal access tokens**. Choose **Tokens (classic)**.

![gh1](https://raw.githubusercontent.com/vottri/CICD-pipeline-with-Jenkins/main/images1/gh1.png)

Click on **Generate new token** button on the next page. Also choose **Generate new token (classic)**.

![gh2](https://raw.githubusercontent.com/vottri/CICD-pipeline-with-Jenkins/main/images1/gh2.png)

Give your token a name in the **Note** bar.

![d5](https://raw.githubusercontent.com/vottri/Docker/main/images/d5.png)

In the **Select Scopes** section, choose the same as below:

![gh3](https://raw.githubusercontent.com/vottri/CICD-pipeline-with-Jenkins/main/images1/gh3.png)
 
When you are finished, click **Generate token** at the bottom.

You get a token.

![gh4](https://raw.githubusercontent.com/vottri/CICD-pipeline-with-Jenkins/main/images1/gh4.png)
 
Copy the token, make a note of it so we can use it later. 

### Get the repository URL

On GitHub.com, navigate to the main page of the repository:

Above the list of files, click **Code**. Copy the URL for the repository.

![d4](https://raw.githubusercontent.com/vottri/Docker/main/images/d4.png)

### Clone your remote repository on Github to create a local copy on your Linux machine to practice Docker

Go back to your Linux machine. Navigate to the location where you want to put your local clone or repository.

Type **git clone**, and then paste the URL you just copied. Add **your github username** and the copied **personal access token** to the command according to the following structure:

```sh
git clone https://your-github-username:personal-access-token@github.com/your-github-username/your-repository.git
```

Moreover, to clone a specific branch of the remote repository on Github, you might want to add the following option.

```sh
git clone -b <branch> <remote_repository_url>
```

I just cloned the **dev** branch of the repository I want on my Github account back to the Linux machine.

![d6](https://raw.githubusercontent.com/vottri/Docker/main/images/d6.png)

## 4. Docker images <a name="4"></a>

Go inside the local git repository that you just cloned.

![d7](https://raw.githubusercontent.com/vottri/Docker/main/images/d7.png)

The developers have already set up dockerfiles for the web app and web api. 

![d8](https://raw.githubusercontent.com/vottri/Docker/main/images/d8.png)

![d9](https://raw.githubusercontent.com/vottri/Docker/main/images/d9.png)

We are going to build the images using those dockerfiles.

### Build web app image

```sh
docker build -f ./DevOpsWeb/Dockerfile -t devopsweb:1.0 .
```

 + **-f**: lets you specify the path to the Dockerfile (The above command will use the current directory as the build context and read a Dockerfile from **./DevOpsWeb/**).

 + **-t**: lets you tag the image (The image name will be **devopsweb** and the tag will be **1.0**).

![d10](https://raw.githubusercontent.com/vottri/Docker/main/images/d10.png)

### Build web api image

```sh
docker build -f ./DevOpsWeb.WebApi/Dockerfile -t devopsweb-api:1.0 .
```

![d11-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d11-1.png)

### Pull database image

The developers didn't provide source code or any dockerfile for building the database image. 

We need to pull pre-built Docker image for Microsoft SQL Server based on Ubuntu back to the Linux machine.

Docker Hub provides a lot of container images for Microsoft SQL Server on Linux at https://hub.docker.com/_/microsoft-mssql-server/. You can go through the list to search for the image you want.
 
When you are ready, pull the image back to your Linux machine

```sh 
docker pull mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04
```
 + **docker pull [image-name]:[tag]**

![d12](https://raw.githubusercontent.com/vottri/Docker/main/images/d12.png)

Remove unused images for efficiency's sake (optional).

```sh
docker image prune -f
```

You now can list the images you have on your machine.
 
```sh
docker image ls
```

![d13](https://raw.githubusercontent.com/vottri/Docker/main/images/d13.png)

## 5. Docker containers <a name="5"></a>

Check for your Linux machine's internal IP address. Sometimes you may need your host's IP address for your Docker container. For example, you need to be able to connect to the host network from inside a Docker container to access your app or database running locally on the host.

![ip](https://raw.githubusercontent.com/vottri/Docker/main/images/ip.png)

### Start a container from the mssql image you pulled

```sh
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" -p 1433:1433 --name mssql-server -d mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04
```

 + **-e**: lets you set environment variables in the container you’re running.

 + **--name**: lets you assign a name to the container (with "mssql-server" being the name in this case).

 + **-p [host-port]:[container-port]**: lets you publish a container's port(s) to the host (-p 1433:1433 map port 1433 from the host (your Linux machine) to port 1433 on docker container).

 + **-d**: lets you run containers in the background.

![d14](https://raw.githubusercontent.com/vottri/Docker/main/images/d14.png)

List the running containers by using one of these commands.

```sh
docker container ls 

docker ps
```

![d15](https://raw.githubusercontent.com/vottri/Docker/main/images/d15.png)

### Start web api container

```sh
docker run -e 'ConnectionStrings__AppDatabase=Server=10.0.0.4,1433;Database=AppDatabase;User Id=sa;Password=zaQ@123456!;' --name devopsweb-api-1 -p 10005:80 -d devopsweb-api:1.0
```

![d16-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d16-1.png)

Verify that web api container is running.

![d17](https://raw.githubusercontent.com/vottri/Docker/main/images/d17.png)

### Start web app container

```sh
docker run -e 'WebApi=http://10.0.0.4:10005' --name devopsweb1 -p 10000:80 -d devopsweb:1.0
```

![d18](https://raw.githubusercontent.com/vottri/Docker/main/images/d18.png)

Verify that web app container is also running.

![d19](https://raw.githubusercontent.com/vottri/Docker/main/images/d19.png)

### Access your containers

Inside **Inbound port rules**, add another rule that allow the VM's port **1433** to be access over the Internet.

![lab4](https://raw.githubusercontent.com/vottri/Docker/main/images/lab4.png)

Use **SQL Server Management Studio (SSMS)** to connect to the mssql-server container on your Linux machine. 

Note: You will need to install SSMS on your own computer. Follow these steps to accomplish that:

 - Go to this link: https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16&tabs=command-line and click on **Download SQL Server Management Studio** for SSMS download. 

 - Once downloaded we will get a .exe file named as **SSMS-Setup-ENU.exe**. Double click on it.

 - Click on **Install** button to install **SQL Server Management Studio (SSMS)** on your system. Just wait for it to finish installing.
 
Once you are done with download and setup. Open **SQL Server Management Studio** and it will let you connect to a Microsoft Sql Server if you provide it with appropriate informations.

![lab5](https://raw.githubusercontent.com/vottri/Docker/main/images/lab5.png)

This is a sample table inside a sample database for learning purpose.

![lab6](https://raw.githubusercontent.com/vottri/Docker/main/images/lab6.png)

Inside **Inbound port rules**, add another rule that allow the VM's port **10000** and **10005** to be access over the Internet.

![lab7](https://raw.githubusercontent.com/vottri/Docker/main/images/lab7.png)

Access your VM's Public IP Address over port **10005** to check for connection toward web api container.

![lab8](https://raw.githubusercontent.com/vottri/Docker/main/images/lab8.png)

![lab9](https://raw.githubusercontent.com/vottri/Docker/main/images/lab9.png)

Access your VM's Public IP Address over port **10000** to check for connection toward web app container.

![lab10](https://raw.githubusercontent.com/vottri/Docker/main/images/lab10.png)

## 6. Dockerfile <a name="6"></a>

Dockerfile is a plain-text document that tells Docker how to build an app and dependencies into a Docker image. This time, we are going to manually create a Dockerfile ourselves.

### Create a dockerfile for building the custom database image

```sh
nano Dockerfile
```

![d20](https://raw.githubusercontent.com/vottri/Docker/main/images/d20.png)

Fill the dockerfile with these contents.

```sh
FROM mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04 AS base

WORKDIR /database

USER root

# Download node exporter and extract it.

RUN wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz \
        && tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz

COPY /CustomDB/script.sh .

CMD ["/bin/bash","-c","./script.sh"]

USER mssql
```

Create a script file that will execute when the custom database container starts running.

```sh
nano script.sh
```

![d21](https://raw.githubusercontent.com/vottri/Docker/main/images/d21.png)

Add these contents to the file.

```sh
#!/bin/bash

# Run node exporter and launch the SQL Server when we start the custom database container.

/database/node_exporter-1.5.0.linux-amd64/node_exporter & \
/opt/mssql/bin/sqlservr
```

Change permission for the **script.sh** file.

![d22](https://raw.githubusercontent.com/vottri/Docker/main/images/d22.png)

### Build the custom database image

```sh
docker build -f ./CustomDB/Dockerfile -t customdb:1.0 .
```

![d23](https://raw.githubusercontent.com/vottri/Docker/main/images/d23.png)
![d24](https://raw.githubusercontent.com/vottri/Docker/main/images/d24.png)

Verify the image after the build.

![d25-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d25-1.png)

### Create a volume for the custom database container

Volumes are the recommended way to persist data in containers. Persistent data is the data you need to keep. Things like: customer records, financial data, audit logs, ...

Basically:

 - You create a volume, then you create a container and mount the volume into it. 

 - The volume is mounted into a directory in the container’s filesystem, and anything written to that directory is stored inside the volume. 
 
 - If you delete the container, the volume and its data still exist.

Create a directory on your Linux machine so you can mount it to the container later. And to prevent permission denied error, change its permission:

```sh
mkdir -p mssqlvolume/data
sudo chown -R 10001 mssqlvolume/
```

### Start the custom database container

```sh
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" --name customdb1 -p 1434:1433 -p 9100:9100 -d -v /home/cloud_user/mssqlvolume/data:/var/opt/mssql/data customdb:1.0
```

 + **-v [host directory]:[the path where the directory are mounted in the container]**: lets you mount host-directories in a container.
 
![d27](https://raw.githubusercontent.com/vottri/Docker/main/images/d27.png)

Verify that the container is running.

![d28](https://raw.githubusercontent.com/vottri/Docker/main/images/d28.png)

### Access your container

Inside **Inbound port rules**, add another rule that allow the VM's port **1434** and **9100** to be access over the Internet.

![lab11](https://raw.githubusercontent.com/vottri/Docker/main/images/lab11.png)

You can check if node exporter is running or not by accessing VM's Public IP Address over port **9100**.

![lab12](https://raw.githubusercontent.com/vottri/Docker/main/images/lab12.png)

Use **SQL Server Management Studio (SSMS)** to connect to your custom database container (this time over port 1434).

![lab13](https://raw.githubusercontent.com/vottri/Docker/main/images/lab13.png)

Create a Test database.

![lab14](https://raw.githubusercontent.com/vottri/Docker/main/images/lab14.png)

![lab15](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15.png)

Check the volume on your Linux machine.

![lab15-1](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15-1.png)

For testing purposes, stop and remove your custom database container.

```sh
docker container stop customdb1 

docker container rm customdb1
```

![lab15-2](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15-2.png)

Your volume data still exists.

```sh
ls -la mssqlvolume/data
```

![lab15-1](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15-1.png)

Start a new custom database container but attach the same volume.

![lab15-3](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15-3.png)

Check for your old Test database by accessing your newly created container.

![lab15](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15.png)

Note: To start with a clean slate and remove all running Docker containers on your machine, there are a set of handy commands:

List all containers (only IDs)

```sh
docker ps -aq
```
Stop all running containers

```sh
docker stop $(docker ps -aq)
```

Remove all containers

```sh
docker rm $(docker ps -aq)
```

![lab15-5](https://raw.githubusercontent.com/vottri/Docker/main/images/lab15-5.png)

## 7. Dockercompose <a name="7"></a>

Docker Compose lets you describe multi-container applications in a single declarative configuration file, and deploy it with a single command. Once the app is deployed, you can manage its entire lifecycle with a simple set of commands. You can even store and manage the configuration file in a version control system.

### Docker Compose files

Docker Compose uses YAML file to define multi-service applications. The default name for a Docker Compose YAML file is dockercompose.yml. (you can use the -f flag to specify custom filename.)

Create a dockercompose.yml file

```sh
nano docker-compose.yml
```

![d29](https://raw.githubusercontent.com/vottri/Docker/main/images/d29.png)

Our specific Docker Compose file

```sh
version: "3.8"    # define the version of the Compose file format (basically the API)

services:         # define different application microservices.
  devops-web:
    image: devops-web:web-1.0
    build:
      context: .
      dockerfile: DevOpsWeb/Dockerfile
    ports:
      - target: 80
        published: 10000
      - target: 443
        published: 10001
    environment:
      - WebApi=http://devops-api
    depends_on:
      - devops-db
      - devops-api

  devops-api:
    image: devops-web:api-1.0
    build:
      context: .
      dockerfile: DevOpsWeb.WebApi/Dockerfile
    ports:
      - target: 80
        published: 10005
    environment:
      - ConnectionStrings__AppDatabase=Server=devops-db;Database=AppDatabase;User Id=sa;Password=zaQ@123456!;
    depends_on:
      - devops-db

  devops-db:
    image: devops-web:db-1.0
    build:
      context: .
      dockerfile: CustomDB/Dockerfile
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=zaQ@123456!
      - MSSQL_PID=Express
    ports:
      - target: 1433
        published: 1434
      - target: 9100
        published: 9100
    volumes:
      - /home/cloud_user/mssqlvolume/data:/var/opt/mssql/data

networks:         # create new networks.
  default:
    name: devops-shared-network
```

### Deploy apps with Docker Compose

**Docker compose up** is the most common way to bring up apps. It builds or pulls all required images, creates all required networks and volumes, and then starts all containers.

```sh
docker compose up -d
```
 + **-d**: lets you bring those app up in the background.
 
![d30](https://raw.githubusercontent.com/vottri/Docker/main/images/d30.png)

Now that the apps are built and running, we can use normal docker commands to view the images, containers, networks that Docker Compose created.

```sh
docker image ls
```
![d31-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d31-1.png)

```sh
docker container ls
```

![d31](https://raw.githubusercontent.com/vottri/Docker/main/images/d31.png)


```sh
docker network ls
```

![d31-2](https://raw.githubusercontent.com/vottri/Docker/main/images/d31-2.png)

Now, with the web app successfully deployed, you can visit a web browser and access your website at port **10000** to check its content.

![lab10-1](https://raw.githubusercontent.com/vottri/Docker/main/images/lab10-1.png)

As for the custom database container, you can check for the running node exporter by accessing your VM's public IP address at port **9100**.

![lab12](https://raw.githubusercontent.com/vottri/Docker/main/images/lab12.png)

Or you can use **SQL Server Management Studio (SSMS)** to connect to your custom database container and view its databases.

![lab16](https://raw.githubusercontent.com/vottri/Docker/main/images/lab16.png)

As you can see, the applications are already up, but now you want to bring them down. To do that, it is quite simple, just replace the **up**  with **down** and let Docker Compose handles the rest.

```sh
docker compose down
```

![d32](https://raw.githubusercontent.com/vottri/Docker/main/images/d32.png)

You can check for images and containers again.

![d32-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d32-1.png)

### Manage apps with Docker Compose

Simulate the change in source code of the web application.

Head over to the web page of the web application.

![d33](https://raw.githubusercontent.com/vottri/Docker/main/images/d33.png)

Try and modify come contents of the file.

![d34](https://raw.githubusercontent.com/vottri/Docker/main/images/d34.png)

You can use **docker compose build [service/app name]** to only build the wep application again.

```sh
docker compose build devops-web
```

![d35](https://raw.githubusercontent.com/vottri/Docker/main/images/d35.png)

Now run docker compose up once more time to bring the apps up.

![d36](https://raw.githubusercontent.com/vottri/Docker/main/images/d36.png)

List your running containers to see whether your apps are already up.

![d37](https://raw.githubusercontent.com/vottri/Docker/main/images/d37.png)

Try accessing your website again.

![lab17](https://raw.githubusercontent.com/vottri/Docker/main/images/lab17.png)



