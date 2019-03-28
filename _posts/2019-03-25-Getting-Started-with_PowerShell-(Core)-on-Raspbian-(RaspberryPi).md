---
layout: post
title: Getting started with PowerShell (Core) on Raspbian (RaspberryPi)
subtitle: Light up a Led
#gh-repo: daattali/beautiful-jekyll
#gh-badge: [star, fork, follow]
tags: [PowerShell, RaspberryPi]
comments: true
---

Around a year ago, I started to investigate about new usages for my Raspberry Pi 3b . I love the Raspberry, a tiny, credit-card-sized SBC capable of running different operating systems, but most of the projects I found that seemed interesting were highly complex (or at least seemed) and used python, a language that I'm not familiar with. Around the same time, I also started to hear a lot about PowerShell (or maybe it caught my attention because I've started to use it at my work for simple stuff), as a scripting language mainly used (as far as I knew) to automate tasks.

## PowerShell Core - A shell for every ecosystem

That's right, [PowerShell Core](https://github.com/PowerShell/PowerShell) (PowerShell from now on) is cross-platform and works on Windows, Linux and MacOS). This is a _HUGE_ step for people that are used to PowerShell but weren't able to do so outside of Windows. Sure, there are still limitations, but the progress has been huge.

## PowerShell.IoT - The module

This module is the main reason why I decided to try PowerShell on Raspberry. A couple of weeks before the release of this module, I had bought a [Scroll pHAT](https://github.com/pimoroni/scroll-phat) (that seems to be no longer for sale) as an incentive to learn python. But after seeing a tweet about the release of [this module](https://github.com/PowerShell/PowerShell-IoT) everything changed! I got hooked with the possibility of controlling LEDs, sensors and much more with only PowerShell!

One of the first things that I've done was to create a module to control the scroll pHAT that I've bought (guess what - I ended up not learning python), which got [featured on Pimoroni's GitHub repo](https://github.com/pimoroni/scroll-phat#alternative-libraries)!

## Enough story, show me what I've came to see!

Alright, if that's what you want, that's what you will get. Let's start by getting PowerShell. Microsoft's documentation here is really good. I will leave the link [here](https://docs.microsoft.com/en-gb/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#raspbian) if you prefer to check their instructions, but I will help you and show the instructions, so that you don't need to jump back and forth:

1. Start by downloading the version that you want to your Raspberry. Remember that you can copy the link and use _wget_ to download the file. (I always download the latest previews, this way I can test and report if I spot something) - [Here's the link](https://github.com/PowerShell/PowerShell/releases). You want to look for something like _powershell-x.x.x-**linux-arm32**.tar.gz_ (by the time of this writing, latest version was _powershell-6.2.0-rc.1-linux-arm32.tar.gz_)
2. Install the pre-requisites `sudo apt-get install libunwind8`
3. Create a folder where you want to store the powershell required files. For example  `mkdir ~/powershell`
4. Unpack the tar file and copy the content to the folder you previously created. `tar -xvf ./powershell-YOUR_VERSION-linux-arm32.tar.gz -C ~/powershell`
5. Congratulations, you can now launch PowerShell by going to the folder where the content is and type `pwsh` (according to this example, that is ​`~/powershell/pwsh`).
6. Optionally, you can create a symlink so that you don't have to specify the full path: `sudo ln -s ~/powershell/pwsh /usr/bin/pwsh`

Alright, so with this you have PowerShell ready to do whatever magic you want!

## PowerShell.IoT module - Installation and usage

If you are already familiar with modules on PowerShell, this one is not different. The only "exception" here is that we will launch and use PowerShell with higher privileges (sudo). Why is that, you might be asking? Well, because this module interacts with GPIO, I2C, SPI, which require higher privileges.

So, start by launching PowerShell with higher privileges - `sudo pwsh`. After that, all we need to do is `Install-Module Microsoft.PowerShell.IoT`. This will ask if you trust the source, you can say you do. In case you have any doubts, the project is [hosted on GitHub](https://github.com/PowerShell/PowerShell-IoT) and it's open source.

After it installs, you can import it with `Import-Module Microsoft.PowerShell.IoT`. Remember that you can always get help with `Get-Command -Module Microsoft.PowerShell.IoT` and `Get-Help <Cmdlet name>`.

## Get-GpioPin Set-GpioPin command

The IoT module uses the [WiringPi](http://wiringpi.com/) GPIO pinout. For further reference, you can use this website in order to easily know the pin numbers.
The gpio is fairly simple to understand: each pin has a number and a value (it also has PullUp/PullDown but we will not talk about that part). The pin number varies according to different implementations (remember that in this case, the one that's used is the WiringPi one), and the value can either be high or low . High corresponds to 3.3V and Low to 0V. That said, the cmdlets related to gpio are really simple to understand and use:

* `Get-GpioPin` - you pass an Id or an array of Ids, and it returns the pin's value;
* `Set-GpioPin` - you pass an Id or an array of Ids and that's the value that you want to set.

## Putting this into practice

To apply what we've just learned, let's do the "hello world" of the hardware: lighting up a LED. There are several types of LEDs, but they usually have one "leg" shorter than the other. The shorter leg is the one that's connected to GROUND (cathode), the other is the one that receives the voltage (anode). If your LED is like mine, they have the same size. Other way to know which is the anode and the cathode is by looking to what's inside the LED. As the following image will show you, you have 2 parts, one bigger than the other.

![redLed](/img/Getting_started_with_PowerShell_Raspberry/red_led.png)

The smaller part is the anode (on the right), and the other is the cathode. With this in mind, we can now try it. To see if your led is working, you can connect it directly to the 5V pin, if it lights up, everything is good, otherwise, make sure you didn't swap the wires.

![setup](/img/Getting_started_with_PowerShell_Raspberry/raspberry_setup.jpg)

(My setup, green  connected to cathode (GROUND), grey is connected to anode (3.3V)

Let's go ahead and plug it to the pin 8 (remember that's for WiringPi, which means it's the second pin on the second row). The LED automatically lighted up, didn't it?! That's because that pin is set to the `High` value by default. Let's confirm this by doing `Get-GpioPin -Id 8`. Now it's time to turn it off, by simply doing `Set-GpioPin -Id 8 -Value Low`. Let's setup a loop that will turn it on and of each second.

```PowerShell
$ledPinNumber = 8
while($true){
  Set-GpioPin -Id $ledPinNumber -Value Low
  Start-Sleep -Seconds 1
  Set-GpioPin -Id $ledPinNumber -Value High
  Start-Sleep -Seconds 1
}
```

Here's the result (it is a video, you have to click on the image):

[![IMAGE ALT TEXT](http://img.youtube.com/vi/dKmIJSE6Zko/0.jpg)](http://www.youtube.com/watch?v=dKmIJSE6Zko "Demo - Using PowerShell to control a Led")


This pretty much it. Pretty simple right? Now that you can control the GPIO, spread wings to your imagination and go do awesome stuff! Want a suggestion? What about : _"If the LED is on, send a notification to my machine"_

What's the output? Something like this:
![notification](/img/Getting_started_with_PowerShell_Raspberry/notification.png)

Want to know how that can be achieved? I'll leave that to [another post]({{ site.baseurl }}{% post_url 2019-03-28-Remote-Sessions-and-Notifications %}).

# Other projects

If you want to check up on what I'm up to, feel free to visit [my GitHub page](https://github.com/DanielSSilva).

Also, if you need any help or simply want to discuss/show me something, hit me up on [twitter](https://twitter.com/DanielSilv9)!

Thank you for reading!