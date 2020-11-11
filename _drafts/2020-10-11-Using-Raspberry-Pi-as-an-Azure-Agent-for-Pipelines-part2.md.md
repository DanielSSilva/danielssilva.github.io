---
layout: post
title: Using RaspberryPi as an Azure agent for Pipelines (Part 2)
subtitle: Deploying agents to multiple machines
tags: [RaspberryPi, DevOps, Azure]
comments: true
---

In the [first part](https://danielssilva.dev/2020-09-28-Using-Raspberry-Pi-as-an-Azure-Agent-for-Pipelines/) of this series, we have seen how can we setup a self hosted agent (in this case on a Raspberry Pi) to work as an agent.
To recap, here's a quick list of things required to do on the target host (this is agnostic to the underlaying operating system):
* Get the latest agent version available (head to the Azure DevOps website > Project settings > Agent Pools > your pool > new agent > download agent)
* Extract the file
* Run the config file and specify:
  * Server URL
  * Your PAT
  * Agent pool
  * Agent name
  * Work folder
* Run the run file or configure as a service.

This is fine and easy enough, if you are targeting just one host.

# Deploying agents to my Raspberry Pi cluster

I have cluster consisting on 4x RaspberryPi4 2GB.
Repeating the previous task 3 more times, would not only consume some time, but would also be error-prone.

While I was studying this topic, I've found that you can also deploy agents to containers (we will cover that on part 3).
You can deploy it to containers by essentially running a script that Microsoft made that will do all of the previous steps for you, and running the config with an "unattended" switch.

That seems a promising way to overcome the burden of having to configure all of those steps again and again.

However, there were some changes that I've had to make to [that script (start.sh)](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1) so that it worked in our case.
Even so, we still have to figure a way to run this script on each host.

## Ansible to the rescue

My latest post was about ["A (very) small introduction to Ansible"](https://danielssilva.dev/2020-11-04-A-very-small-introduction-to-Ansible/) and it was specifically written with this scenario in mind.
If you are not familiar with Ansible, I strongly suggest that you give that post a read, so that the next part doesn't seem (so) magic.

In short: we will use Ansible playbooks to automate the whole previous process.

### The playbook

I will start by leaving here the whole playbook.
As an exercice, read it through and try to understand what it will do.
I will then explain the tasks that might not be so obvious or that have some caveats (hint: almost all of them have caveats)
