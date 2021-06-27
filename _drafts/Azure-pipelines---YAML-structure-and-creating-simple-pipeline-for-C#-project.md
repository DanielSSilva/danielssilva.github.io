---
layout: post
title: Azure pipelines - YAML structure and creating simple pipeline for C# project
subtitle: Deploy faster, with more quality and confidence
tags: [Azure Pipelines, DevOps]
comments: true
---

Azure pipelines can be created through GUI or through a YAML file.
I will be focusing on the YAML instead of GUI for three main reasons:

* It is being favored over the GUI
* Allows to commit the YAML to a source control, thus allowing versioning
* Re-usability (will cover it in another post)

# Schema of the YAML file

[Microsoft docs](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema) has an extensive explaination about the schema and everything that is supported.
This is my go-to when I have questions about schema/features.

We will start with the basics and progess whenever needed.
Here's what I consider to be the "basic" structure of the YAML file:

```YAML

trigger:

pool:

jobs:
    task: 
```