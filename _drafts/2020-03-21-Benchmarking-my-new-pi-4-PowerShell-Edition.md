---
layout: post
title: Benchmarking my new Pi4 - PowerShell Edition
subtitle:
tags: [RaspberryPi, PowerShell]
comments: true
---

After a long time waiting for it, I've finally got my hand on the latest Raspberry Pi board - the RaspberryPi 4, 4GB model.
This blog post is the first out of two, where I intend to perform some benchmarks and try to analyse the results.
Such benchmarks will performed on 32 and 64bits, PowerShell 6 and PowerShell 7.

In order to have some way to compare the performance, I will also run the *_same_* benchmarks on my pi 3b and use the same operating systems.


# Raspberry Pi 4 - What's different
One of the things that caught my eyes at first is that they've released, for the first time, the same model with different RAM specs - 1GB, 2GB and 4GB.
There are two main specs that matter for this case:

* CPU - Base clock of 1.5GHz (3b has 1.2GHz)
* RAM - 1/2/4GB (3b has 1GB) - As stated, I'll be using the 4GB version

NOTE: There are many more improvements such as USB 3.0, "true Gigabit ethernet", etc but those don't matter for our scenario.

# Operating systems and PowerShell version

There are plenty of operating systems that can be used on the Raspberry.
The officially supported one is Raspbian - a 32bit Operating system based on Debian.
Because that might be the most stable and supported OS for the Raspberry, that's the one I'll be using for the 32bit tests.
Also, I'll be using the Lite version, which has no UI, nor the recommended software - hence less CPU/memory used overall.

Because there's no 64bit version of Raspbian, I'll be using [Ubuntu server](https://ubuntu.com/download/raspberry-pi).
Since I want to stick with the stability/support, I'll be using the LTS version - 18.04.4 LTS.

As a bonus, I will also run the same benchmarks on PowerShell 6, so that we can see the improvements (if any) between such versions.

TL:DR
* 32bit - Raspbian Lite
* 64bit - Ubuntu Server
* PowerShell - 6 and 7

# Benchmarks

Since the biggest improvements were on CPU and RAM, that's what I'll try to benchmark.

## Environment
In order for the conditions to be as identical as possible, I'll be using the same case, a fan, and the respective official chargers.
Here's the Pi 4


And here's the Pi 3B


## The code

For the CPU there are two tests:
* Single core testing - using the foreach _without_ parallel
* Multi core testing - using the parallel switch on the foreach

Now all I need to do is to create some workload for the CPUs.
For that, I'll create an array of custom objects, where each object has a set of properties that have some random generated data.
I'll be using the `NameIT` module to generate such data.
Kevin Marquette's ([b](https://powershellexplained.com/)|[t](https://twitter.com/KevinMarquette)) [blog post about this module](https://powershellexplained.com/2018-07-09-Powershell-NameIt-generate-random-data/) was so good and had such good examples that made it easy to create one of my own based on his examples.

The custom object has the following structure:
```powershell
[pscustomobject]@{
	Name 		 = Invoke-Generate "[person]"
	Nickname     = Invoke-Generate "[person]##"
	Phone        = Invoke-Generate "+###-###-###-####"
	Email		 = Invoke-Generate "[syllable][syllable][syllable]@[syllable][syllable].com"
	CreationDate = Invoke-Generate "[randomdate]"
}
```

There will be 3 scripts:
* Foreach parallel _WITHOUT_ throttleLimit
* Foreach parallel _WITH_ throttleLimit
* Foreach "sequential" - meaning, _without_ the parallel switch

The first two allows tests to all cores, and the third one allows to test single core performance.

Let's create an array of 1000 elements, by having 6 "runs" of 200 elements each.
Because the Pi has 4 cores, in theory, if we use the parallel we will have 4 "runs" going at the same time, and the other two start as soon as the previous finish.


## How to


