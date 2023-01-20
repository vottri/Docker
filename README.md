============================================================================================

Contents

[1. Lab Setup](#1)

[2. Installing Docker Engine on Linux machine](#2)

[3. Cloning a repository](#3)

[4. Building Docker images ](#4)



============================================================================================

## 1. Lab Setup <a name="1"></a>

Update your system. 

```sh
sudo apt-get update
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
git clone https://[your-github-username]:[personal_access_token]@github.com/your-github-username/your-repository.git
```

Moreover, to clone a specific branch of the remote repository on Github, you might want to add the following option.

```sh
git clone -b <branch> <remote_repository_url>
```

I just cloned the **dev** branch of the repository I want on Github account back to the Linux machine.

![d6](https://raw.githubusercontent.com/vottri/Docker/main/images/d6.png)




Build Docker containers:

1 web app that connects to the database

web app UI needs variable WebApi={ApiUrl}

1 web api 

web api needs to configure connection string to connect to database

ConnectionStrings__AppDatabase=Server={serverIp};Database=AppDatabase;User Id={user};Password={Password};


1 database container database 

database sqlserver 2019 ubuntu 

sa password: zaQ@123456! 

version: express

has installed node exporter and will automatically run when starting the container

After the container starts, there is an automatic script to download a file on the internet.

The database container must have a volume to store data for backup.

web map port 80 (container) - 10000 (host) api run port 80 (container) - 10005 (host) database port 1433 (container) - 1434 (host)

first use docker command normally, then put in docker compose


web app and web api will be dev setup docker file.



## 4. Building Docker images <a name="4"></a>

cd DevOpsRepo/

build web app image

```sh
docker build -f ./DevOpsWeb/Dockerfile -t devopsweb:1.0 .
```

 + **-f**, **(--file)** lets you specify the path to the Dockerfile (The above command will use the current directory as the build context and read a Dockerfile from **./DevOpsWeb/**).

 + **-t**, **(--tag)** lets you tag the image (The image name will be **devopsweb** and the tag will be **1.0**).

build web api image

```sh
docker build -f ./DevOpsWeb.WebApi/Dockerfile -t devopsweb-api:1.0 .
```

https://hub.docker.com/_/microsoft-mssql-server/  Docker images for Microsoft SQL Server based on Ubuntu

```sh
docker pull mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04
```

```sh
docker image prune -f
```

```sh
docker image ls
```

docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" -p 1433:1433 --name mssql-server -d mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04

docker container ls

ip addr

```sh
docker run -e 'ConnectionStrings__AppDatabase=Server=10.0.0.4,1433;Database=AppDatabase;User Id=sa;Password=zaQ@123456!;' --name devopsweb-api-1 -p 10005:80 -d devopsweb-api:1.0
```

docker run -e 'ConnectionStrings__AppDatabase=Server=10.0.0.4,1433;Database=AppDatabase;User Id=sa;Password=zaQ@123456!;' --name devopsweb-api-1 -p 10005:80 -d devopsweb-api:1.0



docker run -e 'WebApi=http://10.0.0.4:10005' --name devopsweb1 -p 10000:80 -d devopsweb:1.0


FROM mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04 AS base

WORKDIR /database

USER root

RUN wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz \
        && tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz

COPY /CustomDB/script.sh .

CMD ["/bin/bash","-c","./script.sh"]

USER mssql




#!/bin/bash

/database/node_exporter-1.5.0.linux-amd64/node_exporter & \
/opt/mssql/bin/sqlservr

chmod 775 script.sh


docker build -f ./CustomDB/Dockerfile -t customdb:1.0 .





docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" --name customdb1 -p 1434:1433 -p 9100:9100 -d customdb:1.0
