---
layout: post
title: Remote Sessions and notifications
subtitle: Send a notification from one machine to another
tags: [PowerShell, RaspberryPi]
comments: true
---

[On my last post]({{ site.baseurl }}{% post_url 2019-03-25-Getting-Started-with_PowerShell-(Core)-on-Raspbian-(RaspberryPi) %}), in the very end, I've showed you an example of something quite simple that you could do with PowerShell on the RaspberryPi: Show a notification when a LED is left ON.

![](/img/Getting_started_with_PowerShell_Raspberry/notification.png)

In this post I will show you how I've managed to display the notification. This will consist in three sections: 
* How to display a notification
* Enabling remote sessions through SSH 
* The final script to display a notification from one device to another

# Showing a notification

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Because <a href="https://twitter.com/WindosNZ?ref_src=twsrc%5Etfw">@WindosNZ</a> <a href="https://twitter.com/hashtag/BurntToast?src=hash&amp;ref_src=twsrc%5Etfw">#BurntToast</a> module is awesome, I figured that there was something cool that I could do with it! And what&#39;s that? Get current temperature from my <a href="https://twitter.com/Raspberry_Pi?ref_src=twsrc%5Etfw">@Raspberry_Pi</a> and receive the notification on windows10 machine. EVERYTHING using only <a href="https://twitter.com/hashtag/PowerShell?src=hash&amp;ref_src=twsrc%5Etfw">#PowerShell</a> <a href="https://t.co/vJCEaoNMqi">pic.twitter.com/vJCEaoNMqi</a></p>&mdash; Daniel Silva (@DanielSilv9) <a href="https://twitter.com/DanielSilv9/status/1051590684485046272?ref_src=twsrc%5Etfw">October 14, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


The first time I've done something similar to this, I've used the [_BurnedToast_](https://github.com/Windos/BurntToast) to do the job. This time, because I am using Linux and because I want this to cover more scenarios, I'll be using [_PoshNotify_](https://github.com/Windos/PoshNotify). 

I could display the notifcation on the raspberry itself, but because in this case I'm using it headless, I will display the notification on my machine instead. In order to achieve this, I need to do two things:
* Install the PoshNotify module
* Somehow establish connection with the machine and send the information. 

## Install the PoshNotify module

Installing PoshNotify is like installing any other module available on PowerShell gallery. Simply do `Install-Module PoshNotify` and you are ready.
After installing it, to see if it works, let's send a simple notification:
```powershell
Import-Module PoshNotify
Send-OSNotification -Title "This is a test" -Body "It has been successfully installed"
```
If everyting went smoothly, you should see a notification.

![](/img/RemoteSessions_Notifications/notification_test.png)

# Configure SSH
Because we will use a PowerShell module to display the notifications, we need to have PowerShell installed on the machine that will do it. Assuming it is already installed, we need to configure the SSH server so that we can use PowerShell to establish remote sessions. What's needed to achieve this can be found [here for Windows](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-6#set-up-on-windows-machine), [here for Linux](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-6#set-up-on-linux-ubuntu-1404-machine) and [here for MacOS](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-6#set-up-on-macos-machine).


## Invoke-Command cmdlet
In order to test if it's working as expected, we will use Invoke-Command. Let's see what does `Get-Help Invoke-Command` gives us:

```powershell
Invoke-Command [[-Session] <PSSession[]>] [-ScriptBlock] <scriptblock> 
[-ThrottleLimit <int>] [-AsJob] [-HideComputerName] [-JobName <string>] 
[-RemoteDebug] [-InputObject <psobject>] [-ArgumentList <Object[]>] 
[<CommonParameters>]
```

Seems that it allows us to pass an array of sessions and a script block.

### PSSession
What can we do with PSSession? `Get-Help *PSSession*` gives us all the cmdlets with PSSession in the name

![](/img/RemoteSessions_Notifications/Get_help_psSession.png)

**NOTE**: Notice that in this case I've used `*` between the `PSSession` so that I catch everything that has the word `PSSession`.

In this case, seems that `New-PSSession` will do the job

`New-PSSession` allows us to specify a hostname and a username. 
Now that we know how to create a session, let's try and run a command. Before we do anything with the Raspberry, let's try it on our machine.
```powershell
$session = New-PSSession -HostName localhost -Username "daniel"
$scriptBlock = {Get-Process}
Invoke-Command -Session $session -ScriptBlock $scriptBlock
```

If you run this, you should be asked to input your password and after you do it, you should see a list of processes currently running on your machine. We can also see that this worked, because we have a header that is the `PSComputerName` and in this case it is localhost, which matches the session we have just created.

![](/img/RemoteSessions_Notifications/get-process_remote.png)

This is looking good! We can use remote sessions to execute scripts on other machine.

# Test send a notification from Raspberry to our machine
Now that we have sucessfully installed and tested _PoshNotify_ and have configured the SSH on the machine that will receive the notification, let's test everything together - let's send a notification from Raspberry to our machine. This is pretty much the same as we've done above, but instead of localhost, we will use our machine's hostname (I like to use the IP address, since sometimes the computer name is not recognized for some reason I'm not aware of - I also know that this has the downside that sometimes the IP address can change, unless you give it a static address).
Let's jump to the raspberry shell and do the following:

```powershell
pwsh
#Uncomment the following line and change the parameters accordingly
#$session = New-PSSession -HostName "YourPCName or IP" -Username "yourUsername"
#In my case it is as follows
$session = New-PSSession -HostName "192.168.1.69" -Username "Daniel"
$scriptBlock = {
    Import-Module PoshNotify
    Send-OSNotification -Title "Remote notification" -Body "This notification has been sent from antoher computer"
}
Invoke-Command -Session $session -ScriptBlock $scriptBlock
```
Again, you will be asked to introduce the password of the **TARGET** machine. You should now successfully see the notification pop up on your computer.

## The final script
The final script, for this case, is that what you can see on the initial screenshot. I'll leave it here.

```powershell
$session = New-PSSession -HostName "192.168.1.69" -Username "Daniel"
$scriptBlock = {
    Import-Module PoshNotify
    Send-OSNotification -Title "LED" -Body "Someone left the led ON!"
}
if((Get-GpioPin -Id 8).Value -eq 'High'){
    Invoke-Command -Session $session -ScriptBlock $scriptBlock
}
```

# Wrapping it up
Hopefully you were able to successfully follow everything we've seen here. If you have questions/comments/feedback, hit me up on [twitter](https://twitter.com/DanielSilv9) (at least until I have a "comments" section)

# What's next?
As you might have noticed, this script still requires improvement. As it is, it's only working while on current powershell session (because of the session parameter). On a future post, we will see how can we schedule this to run every 1 hour (and we will also change what's displayed to be something more useful). So stay tunned!

Thanks for reading!