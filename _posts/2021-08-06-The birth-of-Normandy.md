---
layout: post
title:  "The birth of Normandy"
author: "Eloy Chang"
categories: markdown
project_title: "random"
excerpt: "My first python package! A framework for data pipelines."
path_image: <img class="header_post_image" src='../assets/img/logo epsilon resp.jpg' alt="" />
---

In several projects I have worked on, I has seen a common issue, which is a need of a  standardized way of coding for all the team, if how all a team members work is similar, it would make it simple for debugging, projects rotation and faster developing, clean code guidelines help in this but is not a complete solution.

This motivates me to create a framework for data pipelines, giving a fixed structure but still, enough flexibility to the team members. Also, building a package is something that always has intrigued me, and wanted to do, so _Normandy_ was born.

_Normandy_ is a python framework for data processing pipelines, this means, is a way for processing data, allowing the team to focus on analyzing and other stuff rather than on implementing, and, in the meantime, add features to optimize the pipeline like parallel processing.  

I chose the name as a reference of one of my favorite video games, do you know which is? (more references in the **readme** file).

### What can I do with _Normandy_?

The main idea is to structure your data pipelines, easily configure your data flows and allow you to reuse code.

So, _Normandy_ let you:

* Create multiple data flows, and share code between them avoiding repeat code.

* Parallel processing at process level, allowing you, for example, to read from several sources or produce several outputs from the same data set simultaneously.

* A standardized form of coding.

* A clean and understandable data pipeline structure.

### How to use it?

_Normandy_ is available on pip so, you may install it with just:

```
pip install normandy
```

Once installed use the create project command to create the basic structure of a _Normandy_ project

```
normandy --create-project -project-path your/project/path
```

This will also create a skeleton of the configuration file named `pipeline_conf.yml` inside the **pipeline** folder, which should looks like this:

```
confs:
  path: your/project/path
  envs:
    dev:
      credentials: dev_credentials_file
    prod:
      credentials: credentials_file

flows:
  my-flow:
    steps:
      extract:
      - extract_data
      transform:
      - transform_data
      upload:
      - upload_data
    tags:
    - default
```

It has 2 sections:

* In the **confs** section just need the project path and definitions for the environments, you are free to add whatever definitions you need to differentiate from each environment.

* In the **flows** section is where you define the data flows.

A flow on _Normandy_ is a complete path that a process must take, it is composed of steps, those steps are processed sequentially, so the steps let you keep a sense of order on your flow, but each step may have as many processes you want and those are processed in parallel.

Let's explain it step by step.

After the **flows** keyword you may list all of the flows you want, those need two things:

* Tags: these are used to select which flows to run with a given command.

* Steps: As said before, these define the succession of each group of processes.

List the steps is not enough, this because you may not want to run all the processes defined in the step, this lets you have several flows using the same steps but each one is actually different from each other, and all of this using the same scripts, so to define a step you must list which processes you want to add, you may do this just listing them, but they are also some extra options you can define at process level, which are:

* Error tolerance: Sometimes exists some process which output is not related with the rest of the process, in these case, if this process fail you may still want that the rest of the flow end properly, if that is the case turning on this setting is ideal, which basically say, if the process fail do not interrupt the rest of the flow.

* Avoid tags: When you have several similar flows which differ in just some processes, then you may just define one flow and use tags to select which processes would run.

The structure defined in the configuration file is making reference to a file structure on the pipeline folder, inside it must be a folder for each step (with the same name of the steps), and inside each of them a python file with the name of each process defined on that step, this python file must have a definition of a process function with a fixed structure:

```
from engine.variables_storage import variables_storage

def process(pipe, log):
    # Configurations
    env_confs = pipe.get_confs()
    var_str = variables_storage()
    log.info("Configuration ready")

    # Code here your process
    # ...

    return
```

You may define any other function or import any other library needed but this function is the one which will run when the process is executed.

Finally you may run your data flows with the run-pipeline command:

```
normandy --run-pipeline -tags my_tag -tags sr2 -env prod
```

I hope this could be a useful tool, and I would really appreciate every comment and suggestions to make this better.

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
