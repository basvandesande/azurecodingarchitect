---
title: "GHAS - How to use CodeQL custom queries?"
subtitle :  "Getting started"
description :  "Troubleshooting the workflow, getting things right"
author: "Bas van de Sande"
date: 2023-02-10T18:58:18+01:00
featuredImage :  "/intro/intro-feature.png"
tags :  
    - Github Advanced Security
    - GHAS
    - C#
    - troubleshooting
    - Code scanning
categories : 
    - general
draft :  true
---

Over the last two years, I have seen a growing awareness when it comes to zero trust computing. When organizations look at zero trust computing, the first thing that comes into mind is getting the infrastructure secure. Assuming breach, ensuring that there are multiple layers of security applied. As an engineer I love seeing this growing awareness. What a lot of organizations seem to forget is that perhaps the most important part of securing its data starts with analyzing the source code of its proprietary software.

Nowadays, most software depends on third party (open source) components, which can have vulnerabilities as well. Furthermore the code that is developed within the organization can contain serious security flaws. Github Advanced Security (GHAS) can help those organizations by giving powerful tools like Dependabot (scan source code and its dependencies on vulnerabilities), secret scanning (scan if there are any secrets embedded in source code and configuration files) and last but not least Code scanning. 

GHAS Code scanning is a tool that can scan your source code on any given rule set at any given occasion (e.g. when pushing or pulling code or at a specific interval). The core of GHAS Code scanning is a framework called CodeQL. CodeQL is a query language that is used to perform static code analysis. The framework supports most popular programming languages out there such as Go, Ruby, C#, CPP, Java, Javascript et cetera and is maintained by Github. 

One of the cool things of GHAS Code scanning is that we can add custom queries to it, to scan our code on certain code constructs (such as empty blocks or unintended public methods). Getting started with getting your own custom CodeQL queries in the code analysis workflow can be very cumberstone. At least for me it was hard, because I was not able to find clear documentation and examples on this. It costed me almost a day to get up and running. In this blog post, I will describe how I got my first custom CodeQL query to work.




