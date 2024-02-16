---
title: "Quick tip: Get in control by using your GitHub context"
subtitle :  "Having some fun"
description :  "Use GitHub context to control your workflows"
author: "Bas van de Sande"
date: 2024-02-16T15:35:43+01:00
featuredImage :  "/ghcontext/ghcontext-feature.png"
tags :  
    - GitHub 
    - Context
    - Workflows
categories : 
    - technical
draft: false
---

GitHib actions and workflows are very powerful by nature. They help me to build and deploy my Azure environments, without having me to do all the tedious and errorprone work. I create a pull request, have it peer reviewed and once approved my environments are provisoned the way I envisioned it. This is also the case at my current assignment for a large organization.
Last year the organization decided that a new change management system (which name I refuse to pronounce) should handle all changes.
This means that every step in the process needs to be documented and approved conform ITIL processes.

This collides with all DevOps principles in which teams are responsible for building and running their own code. Instead we are warped back into time, back to release calendars, back to a situation in which organizations release code a couple of times per year. While in the DevOps world we do many releases per day. **\*sigh\***

Anyway it is the status quo and we have to comply. Engineers wouldn't be engineers if they wouldn't try to find a way to automate the hack out of it, thus finding a way to create automated changes, risk assessments and approvals. 

As we are working with GitHub workflows, we set up a way of working in which we build our specific code in branches, create  pull requests and have the code merged back to the main branch. We have triggers on the workflows that ensure that the code gets validated, built and deployed at every push and pull request. 

This means that each time the code gets merged back to main, a new release is created. In case of a specific label applied on the pull request we want to trigger an specific action that handles the change management proces. For this example I created the label "**test**". 

![Pull Request with specific label](/ghcontext/ghcontext-labeled-pr.png)

This is where the GitHub context comes in handy.

## Github context
In a GitHub workflow, you can use variables and objects of all sorts. One of those objects is the GitHub context, that you can call by using **$github**. The Github context is a rich object that contains all information on the Pull Request, Workflow etc. You can check out the context by echoing it to the display while running the workflow. To see the actual contents you need to convert it to a Json representation, such as in the code below.    


```yaml
jobs:
  draft_release:
    name: Create draft release
    runs-on: ubuntu-latest
    steps:
      - name: Create draft release and label PR
        id: create_draft
        ...

      - name: dump context
        run: echo '${{ toJSON(github.event.pull_request) }}'
```

Atter running the specific job, you can check the contents of the context.

![Show the pull request info ](/ghcontext/ghcontext-echo.png)


In our case we are interested to find all labels attached to the Pull Request. Looking through the Json, I find a labels array, containing multiple labels. To show them I alter my code in the workflow and run it again.   

```yaml
jobs:
  draft_release:
    name: Create draft release
    runs-on: ubuntu-latest
    steps:
      - name: Create draft release and label PR
        id: create_draft
        ...

      - name: dump context
        run: echo '${{ toJSON(github.event.pull_request.labels.*.name) }}'
       

```

![Get specific value(s)](/ghcontext/ghcontext-getvalues.png)


## Get into control
Now that I can access the labels (or other fields), I can do what I want to do!  In my case I want to execute an action only if a particular label is set.
I extend the code in my workflow by placing an `if` condition on a step. In this condition I check if the labels array contains the "**test**" label.  
```yaml
jobs:
  draft_release:
    name: Create draft release
    runs-on: ubuntu-latest
    steps:
      - name: Create draft release and label PR
        id: create_draft
        ...

      - name: dump context
        ...
      
      - name: Do my check
        if: ${{ contains(github.event.issue.labels.*.name, 'test' ) }}
        run: echo "Hello world!"
        
```

I run the workflow again, and see that the action gets executed.


![Use values for control](/ghcontext/ghcontext-conditional-use.png)


I change the condition to check for a label that doesn't exist. And run the workflow again... This time, the action is skipped. 

```yaml
jobs:
  draft_release:
    name: Create draft release
    runs-on: ubuntu-latest
    steps:
      - name: Create draft release and label PR
        id: create_draft
        ...

      - name: dump context
        run: echo '${{ toJSON(github.event.pull_request) }}'
      
      - name: Do my check
        if: ${{ contains(github.event.issue.labels.*.name, 'another test' ) }}
        run: echo "Nothing to see... move on!"
        
```

![Use values for control](/ghcontext/ghcontext-conditional-skip.png)


Such simple mechanism, though so powerful. By working with the context, I gained super powers when it comes to controlling my workflow.  

_Happy times ahead!_