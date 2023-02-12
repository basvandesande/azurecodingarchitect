---
title: "GHAS - How to use CodeQL custom queries?"
subtitle :  "Getting started"
description :  "Troubleshooting the workflow, getting things right"
author: "Bas van de Sande"
date: 2023-02-10T18:58:18+01:00
featuredImage :  "/ghas/ghas-feature.png"
tags :  
    - GitHub Advanced Security
    - GHAS
    - C#
    - troubleshooting
    - Code scanning
categories : 
    - general
draft :  false
---

Over the last two years, I have seen a growing awareness when it comes to zero trust computing. When organizations look at zero trust computing, the first thing that comes into mind is getting the infrastructure secure. Assuming breach, ensuring that there are multiple layers of security applied. As an engineer I love seeing this growing awareness. What a lot of organizations seem to forget is that perhaps the most important part of securing its data starts with analyzing the source code of its proprietary software.

Nowadays, most software depends on third party (open source) components, which can have vulnerabilities as well. Furthermore the code that is developed within the organization can contain serious security flaws. GitHub Advanced Security (GHAS) can help those organizations by giving powerful tools like Dependabot (scan source code and its dependencies on vulnerabilities), Secret scanning (scan if there are any secrets embedded in source code and configuration files) and last but not least Code scanning. 

GHAS Code scanning is a tool that can scan your source code on any given rule set at any given occasion (e.g. when pushing or pulling code or at a specific interval). The core of GHAS Code scanning is a framework called CodeQL. CodeQL is a query language that is used to perform static code analysis. The framework supports most popular programming languages out there such as Go, Ruby, C#, CPP, Java, Javascript et cetera and is maintained by GitHub. 

One of the cool things of GHAS Code scanning is that we can add custom queries to it, to scan our code on certain code constructs (such as empty blocks or unintended public methods). Getting started with getting your own custom CodeQL queries in the code analysis workflow can be very cumberstone. At least for me it was hard, because I was not able to find clear documentation and examples on this. It costed me almost a day to get up and running. In this blog post, I will describe how I got my first custom CodeQL query to work.

## Getting up and running
Enabling Code scanning is straight forward. In the Settings section, choose under Security the option "Code security and analysis". Scroll down to Code Scanning and choose to set up an Advanced workflow.  

![Enable Code scanning](/ghas/ghas-enable.png)

A new screen appears, showing a new workflow called "codeql.yml".  This workflow enables automatic code scanning on every push to and pull from the main branch and operates on a cron schedule for periodic scanning.

![Enable Code scanning](/ghas/ghas-workflow.png)

In this workflow we can embed our own CodeQL queries. Before I come to that part, we need to add a query definition to our repository.


## Add my own custom query
Within my repository I created the following folder structure: 

`./.github/codeql/custom-queries`

![Folder structure](/ghas/ghas-folders.png)

Within the custom-queries folder, I store all my custom queries that I want to add as additional queries to the extensive library GitHub is providing. These queries are tailored to your use cases, in my case I added a check for empty code blocks in my source code.

``` sql
/**
 * @id codeql/custom-queries/redundant-if-statements
 * @name Bas's empty blocks
 * @description Find my empty block statements.
 * @kind problem
 * @tags empty
 *       bas
 */

import csharp

from BlockStmt blk
where blk.isEmpty()
select blk, "This 'if' statement is redundant."
```

In its essentials, a CodeQL statement looks a bit like a SQL expression. I'll explain it in more depth.

In top of the file there is a comment header. This comment header is essential and cannot be omitted. The fields **@id**, **@name** and **@kind** need to be set. The **@kind** indicates how Code scanning should handle the results of the query.

The **import csharp** statement describes what programming language is going to be analyzed. In the background specific libraries are present which are used to do the analysis. GitHub provides a detailed description of the [C# CodeQL Library](https://codeql.github.com/docs/codeql-language-guides/codeql-library-for-csharp/).  

In the example above, my source code will be analyzed for the presence of Block statements (**from BlockStmt blk**), e.g. _if_ statements. For each statement that is present in the source code the "**where blk.isEmpty()**"  will check if there is any code within the statement. If there is no code present, then the **select blk, "This 'if' statement is redundant."** will select the empty statement. 

The "This 'if' statement is redundant." description will be shown as a friendly description to indicate why this piece of code was marked as a potential problem.

## Pack the query 
If we want to use the query in our CodeQL workflow, we need to do one additional thing which is "packing" it. Packing means that the custom queries are being compiled to a binary form that can be executed by the CodeQL engine. The compilation can be done upfront or dynamically during the execution of the workflow. 

In this blog, I'll follow the path of the dynamic compilation which means that I have to provide some meta data for the compiler. The meta data is provided in a file called **qlpack.yml** that has to be present in folder of the custom queries that I provide (note that in a single folder multiple query files can be provided).  

![Folder structure](/ghas/ghas-qlpack.png)

A closer look at the **qlpack.yml** file:

``` yaml
name: basvandesande/csharp-custom-queries
version: 0.0.1
dependencies:
  codeql/csharp-all: "*"
extractor: csharp

```

In the **dependencies** array field, at least the language definition library has to be specified with the minimum version required. In this case "*" means all versions. In order to parse your source code correctly, the **extractor** needs to be set to your language. In my case **csharp**. 

Once this is all set, commit and push the code to the repository.

## Modifying the CodeQL workflow
Open the CodeQL workflow 

![CodeQL workflow](/ghas/ghas-codeqlworkflow.png)

Scroll down to **line 53** and modify this line as follows:

  `queries: +security-and-quality, ./.github/codeql/custom-queries`

The line has turned into a comma seperated line containing multiple library packs.  The "security-and-quality" is the default library GitHub provides. By adding the PLUS (+) in front of it, the presence of this library is ensured. 

The second library is the relative path to the folder in which my queries are stored. GitHub Code scanning is pretty picky about the relative folder structure. This costed my a lot of time because existing examples didn't use a local relative path but referred to an external repository instead.

The CodeQL.yml file should look like this after changing the line:


``` yaml
# For most projects, this workflow file will not need changing; you simply need
# to commit it to your repository.
#
# You may wish to alter this file to override the set of languages analyzed,
# or to provide custom queries or build logic.
#
# ******** NOTE ********
# We have attempted to detect the languages in your repository. Please check
# the `language` matrix defined below to confirm you have the correct set of
# supported CodeQL languages.
#
name: "CodeQL"

on:
  push:
    #branches: [ "main" ]
  pull_request:
    # The branches below must be a subset of the branches above
    branches: [ "main" ]
    paths-ignore:
      - '**/*.md'
      - '**/*.txt'
  schedule:
    - cron: '34 12 * * 0'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp' ]
        # CodeQL supports [ 'cpp', 'csharp', 'go', 'java', 'javascript', 'python', 'ruby' ]
        # Use only 'java' to analyze code written in Java, Kotlin or both
        # Use only 'javascript' to analyze code written in JavaScript, TypeScript or both
        # Learn more about CodeQL language support at https://aka.ms/codeql-docs/language-support

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
        queries: +security-and-quality, ./.github/codeql/custom-queries
             
       
        # If you wish to specify custom queries, you can do so here or in a config file.
        # By default, queries listed here will override any specified in a config file.
        # Prefix the list here with "+" to use these queries and those in the config file.

        # Details on CodeQL's query packs refer to : https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-code-scanning#using-queries-in-ql-packs
        # queries: security-extended,security-and-quality


    # Autobuild attempts to build any compiled languages  (C/C++, C#, Go, or Java).
    # If this step fails, then you should remove it and run the build manually (see below)
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    # ℹ️ Command-line programs to run using the OS shell.
    # 📚 See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun

    #   If the Autobuild fails above, remove it and uncomment the following three lines.
    #   modify them (or add more) to build your code if your project, please refer to the EXAMPLE below for guidance.

    # - run: |
    #     echo "Run, Build Application using script"
    #     ./location_of_script_within_repo/buildscript.sh

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
      with:
        category: "/language:${{matrix.language}}"

```

## Let the Code scanning begin!
Each time I push code to or pull code from 'main', the CodeQL workflow will start processing in the backgound. 

![CodeQL Code scanning](/ghas/ghas-force.png)

Once the code scanning finished successfully, the security risks can be assessed via the "Security" option in the menu bar. The security option will show a small shield containing the number of risks detected.

![CodeQL Security risks](/ghas/ghas-risks.png)

As you can see, I have three locations in my code containing an empty code block. Zooming in:

![CodeQL Security risk zooming in](/ghas/ghas-zoomin.png)

The friendly message that I provided in the query is shown under the highlighted empty code block, giving me a clue on how to handle the alert. Each alert can be dismissed. Once dismissed, the specific alert instance won't be shown anymore in the list.  


With all mechanisms in place, it is time to start working on my own custom queries. 
