---
title: "Note to self: localhost in kestrel is localhost"
subtitle :  "Problems accessing Kestrel website on remote host"
description :  "Port binding under Kestrel does not alway work as expected. You need to specify the IP Address as well..."
author: "Bas van de Sande"
date :  2022-02-23T17:20:25+02:00
featuredImage :  "/kestrel/kestrel-feature.png"
categories: 
    - technical
tags: 
    - C#
    - debug
    - troubleshooting
toc: false
draft: false
---
In one of the projects I'm involved in, I ran into a problem.  In this project we are building a solution that is running on a Linux based Edge Device connected to Azure. On the Edge device a [k3s](https://k3s.io/) cluster hosts a number of services required by the solution. In order to be able to develop and debug on a Linux environment, I equiped my system with Hyper-V, an internal NAT switch and a fresh copy of Ubuntu.

Before starting actual development I created an out-of-the-box ASP.NET Core Web app, and published it as a Linux-X64 package on my Ubuntu image and started the application on the linux machine.

```
cd /home/bas/debugtest
dotnet ./linuxdebugtest.dll
```

![localhost](/kestrel/kestrel-localhost-5000.png)

I opened Chrome on my Windows host and tried to navigate to the test application (bas-edge:5000). Unfortunately I received the not connected message (ERR_CONNECTION_REFUSED).

![not connected](/kestrel/kestrel-oops.png)

 For some reason I was not able to connect to ASP.NET Core application. What really confused me that I was able to attach the Visual Studio debugger to the published webservice on the VM. Samba was working, FTP was working... I was baffled.  *Could it be a port?*  
 
 The first step was to ensure that the ufw (uncomplicated firewall) on ubuntu was disabled. Actually this can be dangerous, as there is no protection againt malicious traffic on the VM. In this case not a problem because the network is an internal network on my machine. 

 ```
sudo ufw disable
 ```

Then I opened the appsettings.json file, but nothing was to be seen there. 

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

I did some googling and found an article that you could specify to what ports Kestrel (the embedded NET CORE webserver) should be using. I extended the appsettings.json with the following lines to make kestrel listening to port 5400 (http) and 5401 (https) on Localhost.

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5400"
      },
      "Https": {
        "Url": "https://localhost:5401"
      }
    }
  }
}

```
I restarted the webserver and saw that it was listening to port 5400/5401

![localhost](/kestrel/kestrel-localhost.png)

And I tried again without luck. It turned out that Kestrel is very picky about the urls you specify. When you specify localhost, Kestrel will only listen to the localhost IP Address (which is 127.0.0.1). In Kestrel you need to specify the IP address on which it should be listening. This is either the IP address of the machine of even better a wildcard (*).

Changing the url to  http://*:5400 and https://*:5401 solved my issue...

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://*:5400"
      },
      "Https": {
        "Url": "https://*:5401"
      }
    }
  }
}

```

![](/kestrel/kestrel-connected.png)

A lesson to remember, time for serious development...
