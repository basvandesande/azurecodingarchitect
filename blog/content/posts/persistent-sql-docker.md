---
title: "Running a persistent SQL Server in Docker"
subtitle :  "Make old skool development cool again."
description :  "SQL Server running in Docker with persistent storage and automatic rebinding after startup."
author: "Bas van de Sande"
date :  2022-03-05T17:46:30+02:00
featuredImage :  "/sqldocker/sqldocker-feature.png"
tags :  
  - Docker
  - SQL Server 
  - Csharp
categories :  
  - development
draft :  false
---
In IT we always want to work with the latest and greatest, it's in our DNA to explore new technologies. From time-to-time you get confronted with legacy technologies, robust but boring. One of such things is a piece of software I wrote more than 15 years ago for a friend. It was an application using a SQL Server database that controlled a couple of weighing terminals connected over RS232. Controlling machines is every geek's dream.
Every couple of years my friend asks me if I can help him to improve the software I wrote for him, this time it was no different. 

The tedious part of doing extensions to old software is that you have to set up an environment to do so. Visual Studio is always present on my System. Source Code can be easily retrieved from the repository. Databases can be reattached to the database server. Well wait... what database server? My system has no database server installed and frankly I'm not very willing to install a database server on my Windows system. Database Servers - such as SQL Server - are known memory and resource hogs.

This time I wanted to do it a bit different. I remembered that at Ignite 2019, Microsoft publically introduced SQL Server for Linux. Since I have WSL2 and Docker Desktop running on my system, why not run SQL Server from within a Docker Container? That way I only have to mount my database and ensure that the data modifications are persisted.  Not rocket science, but a fun and convenient twist to this legacy project.

#### My wishes
Using Docker I wanted to achieve the following: 
1. SQL Server has to be available on demand. When I don't need it, it shouldn't be consuming resources (besides storage).
2. All actions in the databases should be persisted, which means that data or scheme modifications should be available next time I start the SQL Server docker container
3. It should be extendable without much effort. With extendable I mean that I easily can add or remove databases if needed.

#### Docker Compose
In the root of my project I added the following docker-compose.yaml file. 

```
version: '3.7'
services:
  SQLServer:
   image: mcr.microsoft.com/mssql/server:2019-latest
   container_name: SQLServer2019
   environment:
     - ACCEPT_EULA=Y
     - SA_PASSWORD=MyVeryVerySecretPassword   
     - attach_dbs=[
          {
            'dbName':'WeegApplicatie',
            'dbFiles': [ '/var/opt/mssql/data/weegapplicatie.mdf' , '/var/opt/mssql/data/weegapplicatie_log.ldf']
          }
       ]
   ports:
     - '1433:1433'
   volumes:
     - C:\Data/SQLServerData:/var/opt/mssql/data
```

In the Docker Compose I do the following: 

1. Take the SQL Server 2019 image from the container registry. 
2. Set the environment variables needed for the initialization of SQL Sever (acceptance of the EULA and the SA password)
3. Attach the database files that I need. Therefor specify a Database name that will be shown in SQL Server Management Studio and the name of  the SQL data file and SQL transaction log file.
In the attach_db section, as many databases can be attached as needed.
4. Specify the port number for the container. In this case I map the well known port 1433 from the host to the image.
5. Finally specify the folder location on the host system (c:\Data/SQLServerData) and the path that is used inside the docker image (/var/opt/mssql/data).

#### Let's use it 

The docker-compose is added to Docker by entering the following command on the command prompt from within the folder where the docker-compose.yaml is located:

```
docker-compose up
```


When opening Docker Desktop, the SQL Server docker is already started.

![Docker Desktop](/sqldocker/sqldocker-docker.png)

When opening SQL Server Management Studio for the first time, the database needs to be attached. This is a one time action. The next time SQL Server is started the database will be attached manually; amd can be accessed directly.

![SQL Server Management Studio](/sqldocker/sqldocker-ssms.png)

**Sweet!** 

Just open Docker Desktop and start the SQL Server image when you need it. When you don't need your database server anymore, all you have to do is to stop the image. SQL Server doesn't get much easier and more convenient than this. 

