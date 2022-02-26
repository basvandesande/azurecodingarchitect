---
title: "Running a persistent SQL Server in Docker"
subtitle :  "Make old skool development cool again."
description :  "SQL Server running in Docker with persistent storage and automatic rebinding after startup."
author: "Bas van de Sande"
date :  2022-02-25T12:46:30+02:00
featuredImage :  "/sqldocker/sqldocker-feature.png"
tags :  
  - Docker
  - SQL Server 
  - C#
categories :  
  - development
draft :  true
---
In IT we always want to work with the latest and greatest, it's in our DNA to explore new technologies. From time-to-time you get confronted with legacy technologies, robust but boring. One of such things is a piece of software I wrote more than 15 years ago for a friend. It was an application using a SQL Server database that controlled a couple of machines connected over RS232. Controlling machines is a geek's dream.
Every couple of years my friend asks me if I can add some features or improvements to the software, this time it was no different. 

The tedious part of doing extensions to old software is that you have to set up an environment to do so. Visual Studio is always present on my System. Source Code can be easily retrieved from the repository. Databases can be reattached to the database server. Well wait... what database server? My system has no database server installed, and frankly I'm not very willing to install a database server on my Windows system. Database Servers - such as SQL Server - are memory and resource hogs.

This time I wanted to do it a bit different. I remembered that at Ignite 2019, Microsoft publically introduced SQL Server for Linux. Since I have WSL2 and Docker Desktop running on my system, why not run SQL Server from within a Docker Container. That way I only have to mount my database and ensure that the data modifications are persisted.  Not rocket science, but a fun and convenient twist to this legacy project.

I set a couple of requirements: 
1. SQL Server has to be available on demand. When I don't need it, it shouldn't be consuming resources.
2. Databases should be persisted, which means that data or scheme modifications should be available next time I start SQL Server
3. It should be extendable without much effort. With extendable I mean that I easily can add database if needed.

