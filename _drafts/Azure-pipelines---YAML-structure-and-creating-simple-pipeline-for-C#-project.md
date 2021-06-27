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

Let's start with the concepts.
Here's what I consider to be the "basic" structure of the YAML file:

```YAML
trigger:
pool:
jobs:
  - job:
    steps:
    - task:
    - task:
  - job: 
    steps:
    - task:
    - task:
```

Let's break it down into what each element does:

* *trigger*: as the name implies, here you configure what will trigger your pipeline. The [documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#triggers) has a lot of information, but in short, there are 4 trigger types: Push, Pull Request, Scheduled and Pipeline. Because this point itself can be a blog post, I will dedicate one just for this. For now, we can set this to "none", meaning it will never be triggered (you have to call it manually).
* *pool*: Defines which pool the pipeline should run on. You can run on a Microsoft-Hosted pool or on a private pool. For new we will use the Microsoft-hosted pool, but on a future post we will see how to create our own, but if you are really curious, you can check my [Using RaspberryPi as an Azure agent for Pipelines (Part 1)](https://danielssilva.dev/2020-09-28-Using-Raspberry-Pi-as-an-Azure-Agent-for-Pipelines/) post.
* *jobs*: The pipeline can have multiple jobs. Unless stated otherwise (by using the dependsOn parameter), jobs run in parallel. Each job has *steps*, which are defined into one or many *tasks*

# Project strucute

For the sake of simplicity, let's say we have a simple C# console application that outputs a greeting message given the user name as parameter.
You can find this project on my Azure DevOps project repository called [AzDevOpsSeries](https://dev.azure.com/danielssilvadev/_git/AzDevOpsSeries) 
This message is given by calling the "GreetUser" method:
```csharp
public static string GreetUser(string name)
{
    return $"Hello {name}, welcome!";
}
```

So, if we run `dotnet run "Daniel"`, it outputs:

```
Hello Daniel, welcome!
```

Now, because we don't like broken things, we add a simple unit test to make sure that the method is returning what's expected.

```csharp
[Fact]
public void Test1()
{
    string userName = "SomeUser";
    string expected = $"Hello {userName}, welcome!";
    string actual = GreeterApp.Program.GreetUser("SomeUser");
    
    Assert.Equal(expected, actual);
}
```

Running `dotnet test` we get that:
```
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: < 1 ms - GreeterTests.dll (net5.0)
```

So our project structure is the following:
```
GreeterProject
|
|-----GreeterApp
|  |
|  |-----GreeterApp.csproj
|
|-----GreeterTests
|  |
|  |-----GreeterTests.csproj
```

# Creating our pipeline
It's now time to create our pipeline.
Because we want to keep it in source control, let's keep it "near" our project.

```
GreeterProject
|
|-----GreeterApp
|  |
|  |-----GreeterApp.csproj
|
|-----GreeterTests
|  |
|  |-----GreeterTests.csproj
|
|-----Pipelines
|  |
|  |-----Greeter.yaml
```

So the idea for our pipeline is to restore, build , test and publish our GreeterApp 