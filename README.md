============================================================================================

Contents

[1. Lab Setup](#1)

[2. Installing Docker Engine on Linux machine](#2)

[3. Cloning a repository](#3)

[4. Docker images ](#4)

[5. Docker containers ](#5)

[6. Dockerfile ](#6)

[7. Dockercompose ](#7)

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
git clone https://your-github-username:personal-access-token@github.com/your-github-username/your-repository.git
```

Moreover, to clone a specific branch of the remote repository on Github, you might want to add the following option.

```sh
git clone -b <branch> <remote_repository_url>
```

I just cloned the **dev** branch of the repository I want on my Github account back to the Linux machine.

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

You now can list the images you have on your machine.
 
```sh
docker image ls
```

![d13](https://raw.githubusercontent.com/vottri/Docker/main/images/d13.png)


Remove unused images for efficiency's sake (optional).

```sh
docker image prune -f
```

## 5. Docker containers <a name="5"></a>

Start a container from the mssql image you pulled.

```sh
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" -p 1433:1433 --name mssql-server -d mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04
```

 + **-e**: lets you set environment variables in the container you’re running.

 + **--name**: lets you assign a name to the container (with "mssql-server" being the name in this case).

 + **-p [host-port]:[container-port]**: lets you publish a container's port(s) to the host (-p 1433:1433 map port 1433 from the host (your Linux machine) to port 1433 on docker container).

 + **-d**: lets you run containers in the background.

![d14](https://raw.githubusercontent.com/vottri/Docker/main/images/d14.png)

List the running containers by using these command.

```sh
docker container ls 

docker ps
```

![d15](https://raw.githubusercontent.com/vottri/Docker/main/images/d15.png)

Start web api container

ip addr

```sh
docker run -e 'ConnectionStrings__AppDatabase=Server=10.0.0.4,1433;Database=AppDatabase;User Id=sa;Password=zaQ@123456!;' --name devopsweb-api-1 -p 10005:80 -d devopsweb-api:1.0
```

![d16-1](https://raw.githubusercontent.com/vottri/Docker/main/images/d16-1.png)

Verify that web api container is running.

![d17](https://raw.githubusercontent.com/vottri/Docker/main/images/d17.png)

Start web app container

```sh
docker run -e 'WebApi=http://10.0.0.4:10005' --name devopsweb1 -p 10000:80 -d devopsweb:1.0
```

![d18](https://raw.githubusercontent.com/vottri/Docker/main/images/d18.png)

Verify that web app container is also running.

![d19](https://raw.githubusercontent.com/vottri/Docker/main/images/d19.png)

## 6. Dockerfile <a name="6"></a>

Create a dockerfile for building the custom database image.

![d20](https://raw.githubusercontent.com/vottri/Docker/main/images/d20.png)

Fill the dockerfile with these contents.

```sh
FROM mcr.microsoft.com/mssql/server:2019-CU18-ubuntu-20.04 AS base

WORKDIR /database

USER root

RUN wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz \
        && tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz

COPY /CustomDB/script.sh .

CMD ["/bin/bash","-c","./script.sh"]

USER mssql
```

Create a script file that will run when the custom database container starts running.

![d21](https://raw.githubusercontent.com/vottri/Docker/main/images/d21.png)

```sh
#!/bin/bash

/database/node_exporter-1.5.0.linux-amd64/node_exporter & \
/opt/mssql/bin/sqlservr
```

Build the custom database image.

![d22](https://raw.githubusercontent.com/vottri/Docker/main/images/d22.png)

```sh
docker build -f ./CustomDB/Dockerfile -t customdb:1.0 .
```

![d23](https://raw.githubusercontent.com/vottri/Docker/main/images/d23.png)
![d24](https://raw.githubusercontent.com/vottri/Docker/main/images/d24.png)

Verify the image after the build.

![d25](https://raw.githubusercontent.com/vottri/Docker/main/images/d25.png)

Create a volume

```sh
mkdir -p mssqlvolume/data
sudo chown -R 10001 mssqlvolume/
```

Start the custom database container.

```sh
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=zaQ@123456!" -e "MSSQL_PID=Express" --name customdb1 -p 1434:1433 -p 9100:9100 -d -v /home/cloud_user/mssqlvolume/data:/var/opt/mssql/data customdb:1.0
```

![d27](https://raw.githubusercontent.com/vottri/Docker/main/images/d27.png)


![d28](https://raw.githubusercontent.com/vottri/Docker/main/images/d28.png)

ls -la mssql-server/data/

Microsoft SQL Server Management Studio 

docker container stop customdb1 

docker container rm customdb1


## 7. Dockercompose <a name="7"></a>

cd DevOpsRepo/

nano docker-compose.yml

```sh
version: "3.8"

services:
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
    image: customdb1:1.0
    build:
      context: .
      dockerfile: ./SqlDockerfile.txt
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
      - /home/cloud_user/mssql-server/data:/var/opt/mssql/data

networks:
  default:
    name: devops-shared-network
```

```sh
docker compose up -d
```

![d30](https://raw.githubusercontent.com/vottri/Docker/main/images/d30.png)

![d31](https://raw.githubusercontent.com/vottri/Docker/main/images/d31.png)

```sh
docker compose down
```

![d32](https://raw.githubusercontent.com/vottri/Docker/main/images/d32.png)

![d33](https://raw.githubusercontent.com/vottri/Docker/main/images/d33.png)

![d34](https://raw.githubusercontent.com/vottri/Docker/main/images/d34.png)

```sh
docker compose build devops-web
```

![d35](https://raw.githubusercontent.com/vottri/Docker/main/images/d35.png)

![d36](https://raw.githubusercontent.com/vottri/Docker/main/images/d36.png)

![d37](https://raw.githubusercontent.com/vottri/Docker/main/images/d37.png)
