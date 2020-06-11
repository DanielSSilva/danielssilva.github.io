---
layout: post
title: Automating database backups with Universal Automation and dbatools 
#gh-repo: daattali/beautiful-jekyll
#gh-badge: [star, fork, follow]
tags: [PowerShell, SQL]
comments: true
---

I've recently read about [Universal Automation (UA)](https://ironmansoftware.com/universal-automation/), by [Ironman Software](https://ironmansoftware.com/).
Among many other functionalities, it offers you the possibility to run background jobs and schedule your scripts to run at certain times.
And that's exactly what we will be exploring today.

# Before we start - Let's understand the product naming

This might sound irrelevant for now, but you will understand in a minute why I'm mentioning this.

The first time I heard about an Ironman Software product it was the [Universal Dashboard (UD)](https://docs.universaldashboard.io/).
In the meantime, another product surfaced: Universal Automation.

When searching for these products, you will find a note saying that these products (UA and UD) are part of something called [PowerShell Universal](https://ironmansoftware.com/powershell-universal/).

## So what is Universal?

Universal is, at the time of this writing, a combination of 3 products:
* [Universal API](https://docs.ironmansoftware.com/api/about) - "(...) provides the ability to define REST API endpoints using PowerShell"
* [Universal Automation](https://ironmansoftware.com/universal-automation/) - "Universal Automation is the ultra-fast, simple, and lightweight automation platform for PowerShell."
* [Universal Dashboard](https://ironmansoftware.com/powershell-universal-dashboard/) - "Build web pages with PowerShell. No HTML or JavaScript required."

This means that Universal is the way to go. From my understanding there won't be any maintenance (bug fixes, features, etc) on each individual product, only on the Universal itself.

## "If that's the way to go, why are you showing Universal Automation?"

At the time of this writing, the current Universal version is the 1.2.0 and I cannot get it to run on my Ubuntu 20.04.
The docs state that, for Linux, the validated distribution is Ubuntu 18.04, so hopefully in the next version I will be able to run it.

My first thought was that I could probably create a Ubuntu 18.04 container, install Universal and run it from a container. But... I'm not familiar with that part of Docker.

Doing a quick search, I've found that they provide a [Docker image for Universal Automation](https://hub.docker.com/r/ironmansoftware/universalautomation/tags).
Well, it's not Universal, but since I'm only interested in UA, I'll take that.
What you will see here, won't be much different from what you can see in Universal.

# Installing Universal (Automation)

We've already seen that I will use the UA docker image in order to run it.
If you are able to use Universal, you can follow the [Getting Started docs](https://docs.ironmansoftware.com/getting-started).

## Running UA on docker

In order to run UA, all we need to do is pull the image by doing `docker pull ironmansoftware/universalautomation:1.3.1`.

When it finishes, we can run it by doing `docker run --name UA -d -p 8080:8080 -p 10001:10001 ironmansoftware/universalautomation:1.3.1`.

In case you are not familiar with docker, this command will:

* Create a container with an `ironmansoftware/universalautomation:1.3.1` image;
* Give the container the name specified after the `--name` switch (`UA` in this case)*;
* Run the container in detached mode (`-d`). This allows us to keep using the terminal;
* Map the ports (with the `-p` switch) 8080 on the container to the port 8080 on the host (our machine). Same for the port 10001;

*__HINT__: I always like to give the containers a name, so that it can be easier to stop, remove, etc

We can then confirm that UA is running by launching `http://localhost:8080/` on our browser.

I recommend that you check [Introducing Universal Automation](https://www.youtube.com/watch?v=u9Hq4X8V7VY) so that you get familiar with the environment and understand the capabilities.

# Some concepts

Let me quickly introduce you some concepts that will be used here:

* Script - That's the same as your PowerShell scripts. It's a piece of code that can be executed.
* Job - Has the responsibility to execute the script and report the execution result such has errors, progress, and so on.
* Schedule - Will run a job at a specified time

# Creating a script 

Once we open the UA, we can create a new script

![new script](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/New_Script.png)

We are asked for some information about the script.
After filling that, we jump to the script and start editing it.

## First attempt

Here's my first attempt to backup a database:

```PowerShell
# BackupDatabase
$secpasswd = ConvertTo-SecureString "Str0ngPassw0rd" -AsPlainText -Force 
$credentials = New-Object System.Management.Automation.PSCredential ("sa", $secpasswd) 
$backupName = "$(Get-Date -Format yyyy_MM_dd_HH_mm)"

Backup-DbaDatabase -IgnoreFileChecks -SqlInstance "192.168.1.80" -SqlCredential $credentials  -database "demo" -BackupDirectory "/backup" -BackupFile "$backupName.bak"
```

After saving and running it, there's an error:

![Backup error](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/BackupError.png)

Well, of course it is not recognized. This is a [dbatools](https://dbatools.io/) command, and since we are running on a container that does not have dbatools installed by default, we have to install it.

## Installing dbatools on the container

To do that, we need to access the container. This can be done by executing `docker exec -it UA /bin/bash`.

( See why it is good to give a name to your container? :) )

Then, launch PowerShell via terminal (`pwsh`) and do `Install-Module dbatools`.

Running the same script again on UA, it now runs successfully!

![Successful Backup](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/Successful_backup.png)

**Bonus tip** : you can run this "all-in-one" command which will launch Powershell and install dbatools `docker exec -it UA /usr/bin/pwsh -interactive -command Install-Module dbatools` 

# Scheduling the backup script

Let's consider that this script has to execute every day, twice a day: at 10 am and at 6 pm.

Going back to our script, we can find a schedule option inside the 3 dots on the top right.

![Schedule](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/Schedule.png)

There are many options baked in it already, such as running every minute, every hour, etc.
In this case, none of those options match our criteria.
But there's another tab called CRON.
That's the one that will be used.
For that, let's create two schedule entries, like this:

![schedule cron](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/schedule_cron.png)

After creating as many schedules as we want, we can see all our schedules by going to the schedules tab.

![schedule list](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/schedule_list.png)

With this, we have our backup script running everyday at 10 am and 6 pm, without us having to remember that we need to execute the script.

# Viewing jobs executions and results

You can easily check the jobs that ran (either by schedule or manually), what's the result, errors, etc.

Here's an example of a script that I've scheduled to run every minute:

![schedule results](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/schedule_sample.png)

And the results can be seen by clicking on the `view` button.
The result looks like this:

![job result](/img/Automating_database_backups_with_Universal_Automation_and_dbatools/Job_result.png)

# Bonus - Leveraging the Secrets

If you navigate to the `variables` tab, you can see that you can add variables.
This is cool if you have multiple scripts that have the same configuration (paths, names, etc).

But what's also cool is that UA allows you to use secrets, meaning you can store your database credentials and avoid placing them as plain text on your scripts.
Unfortunately, I wasn't able to make it work on the version that I'm using.
I see no errors, it says the secret is created successfully, but I can't see it or use it.

I expect that this is something going wrong on my side and that it's working as expected in Universal.
I'll update this post if I can get it working.


# Wrapping up

With this post we've seen what's Universal Automation, Universal and the different naming/functionalities.

We've then used UA through a Docker container, created a script to backup our database and scheduled it to run twice a day, every day.

Hopefully this was a good introduction for you to understand the power of Universal and how you can leverage from it to make your work easier.

On a side note, something that I've found really amazing is that UA was running in a container, and my database is on another container, and it simply worked out of the box!

Thanks for reading!