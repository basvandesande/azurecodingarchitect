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
More than 15 years ago I developed an C# application for a small paint manufacturer, that is being used for administering paint recipes and controlling machines used for paint production. The application relies on central SQL Server for its data storage and multiple weighing terminals  using the data. Every couple of years I receive a request from the company to add or improve some features. 