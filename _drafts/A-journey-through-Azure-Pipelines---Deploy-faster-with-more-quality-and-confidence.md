---
layout: post
title: A journey through (Azure) Pipelines
subtitle: Deploy faster, with more quality and confidence 
tags: [Azure Pipelines, DevOps]
comments: true
---

As a first and most broad idea of what a pipeline is, like to think of it as the filler between developing and delivering a your software. A "piece" that aggregates and performs every necessary action/proceadure (or at least most of them) to take your code up to the final product (an app, a library, a website, you name it).
The most amazing part is that it can be as simple as performing a build to ensure that your code compiles, and grow as your needs, ending up being something like:

build -> test -> generate test report -> deploy to testing environment -> deploy to production -> notify stakeholders through email 

## Pipelines have many different uses

Notice that at the beginning I said "first and most broad idea".
The reason why I said it is that pipelines can be used for things other than build/deploy.
You can use a pipeline to run on a PR and change the state of a workitem of your board, to send an email whenever there's a PR.
You can also use a pipeline to generate infrastucture, manage your git code (create PRs, merges, etc) and many other things.


## My goal

My goal with this series of posts (a journey) is not only to show you how can you setup a simple pipeline for your project(s), but also to keep showing you how can you grow and some of the amazing things that can be done through pipelines.

This blog post can be seen as the intro and entrypoint to check for everything that I write about pipelines (mostly azure pipelines)


# References

* Azure pipelines - YAML structure and creating simple pipeline to build and test C# project