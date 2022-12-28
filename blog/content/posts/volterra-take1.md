---
title: "Project Volterra - Windows Dev Kit 2023 - take 1"
subtitle :  "Volterra in the wild"
description :  "Quirky but fun"
author: "Bas van de Sande"
date :  2022-12-28T13:20:25+02:00
featuredImage :  "/volterra1/volterra-feature.png"
categories: 
    - development
    - technology
tags: 
    - getting started
    - troubleshooting
    - installation
    - usage
toc: false
draft: false
---
At [Xpirit](https://xpirit.com) we have multiple times per year an [Innovation Day](https://youtu.be/5-RDUK7dXvg). The idea of an Innovation Day is to work on technologies or topics that sparked your interest. Anything is allowed, as long as it has nothing to do with any customer assignments. Learning and sharing is the goal of such day. 

At our last Innovation Day in november, I picked up the idea to get some hands on experience with the Windows Dev Kit 2023 also known as Project Volterra. Technology minded as we are, we purchased  two of these boxes at the office for experimentation (big shout out to @marcelv).

### What is this Volterra Windows Dev Kit thingie?
The Windows Dev Kit 2023 is a very small ARM based computer in the shape of a Mac Mini, albeit the Microsoft version is a bad ass black colored machine; using recycled materials for its casing. 

The heart of the machine is an octocore Qualcomm Snapdragon 8cx processor combined with 32GB of DDR4 RAM, a very snappy 512GB ssd and all modern ports and connectivy options that you could wish for. All this goodness comes with a price tag of $599. On top of it, the machine is equiped with a 7 core GPU and a special NPU (neural processor unit) which can be used after obtaining the required drivers from Qualcomm. 

Out of the box it runs Windows 11 for ARM64 and you can download special ARM64 based versions of tools like Visual Studio 2022 for it. Unlike some of the other ARM based Windows machines on the market, this one feels really snappy and runs alomst anything you can throw at it.

![Doom in 100% C#](/volterra1/volterra-doom.jpg)
During the Innovation Day, my colleague Wesley Cabus took his [doom-sharp](https://github.com/wcabus/doom-sharp) project that he built from scratch using C# .NET 6. We expected that it would take some time to get the game running on the Dev Kit, but in fact after a couple of minutes, he was blasting through the first level of the game. **Impressive** 

During the rest of the day we experimented with Kubernetes and managed to edit and render a 4K video using [Filmora](https://filmora.wondershare.com/video-editor-t0.html). In this sense the Windows Dev Kit was nothing more than what you could expect from a normal Windows 11 desktop machine.  

![Desktop](/volterra1/volterra-desktop.jpg)

So far we only experimented with normal use cases. A couple of weeks later my daughter talked to me about a project that she was working at university (Data Science Msc). She told me that she was using Google compute services to train an AI model, because her computer took forever to process through 120 epochs (in Machine Learning an epoch is a complete training dataset) containing over 30GB in images. 

I told her thaţ at the office we had these Developer kits and that I would take one home to see if we could run the workload on the edge.

At home I enabled WSL2 on the machine, installed Python 3.9 on the WSL2 environment, and added all required Python pip packages. I fired up a Jupyter notebook and used the browser to access it. I started to run the algoythms and soon we found out that running a single epoch took 11 minutes using the CPU for processing. Upgrading to Python 3.11 (after compiling a version myself) didn't bring any noticable speed improvements.

On [MS Learn](https://learn.microsoft.com/en-us/windows/arm/dev-kit/) Microsoft mentions that one can unlock the Neural Processor using the ONNX runtime. I descided to give it a go but ran into a wall immediately because [Qualcomm](https://onnxruntime.ai/winarm) needs to give you approval to access the processor. I completed the application form with a solid reason why I need to have access to the processor. Until now, it's crickets....

### This is where I get annoyed
I understand that Microsoft wants developers to jump on the ARM64 bandwagon, and that as a developer you need to buy a special hardware kit to experiment with it to experience its potential. 

What I don't understand is that I as a developer do not get access to one of its (most compelling) core features, which is the Neural Processing Unit. Without having access to this processor, I could have done the Windows ARM64 experiment on a virtual machine. 

As I'm pretty persistent and flexible in the things I do, I decided to go for the second best option: using the GPU. I uninstalled the tensorflow pip package and installed the tensorflow-gpu package. This package would allow me to use the GPU of the machine (7 cores). Soon it turned out that the [GPU is not accessible in WSL2](https://github.com/microsoft/wslg/issues/882) on the ARM64 platform. For me this is a real deception. 

### A mixed bag
So far, the Windows Dev Kit 2023 - Volterra - experience is a mixed bag. The device is snappy, good looking and snappy and I really would love to see fast ARM based Windows devices on the market. It is not as fast as the Apple M1 processor but for now I would settle with that. 

What I don't understand is why Microsoft releases a developer product with its core features inaccessible for now.  In the days to come, I'll see if I can go the Windows route, thus install python on top Windows and find out if I can use the GPU from there. In the meanwhile I keep on hoping that the NPU will be unlocked in the near future.

