---
layout: post
title: Make your ws28xx LEDs smarter by integrating them with google assistant
subtitle:
tags: [RaspberryPi]
comments: true
---

This blog post comes as a result of a couple of things: 
* I've got a Google nest for christmas and wanted to use it for more than the basic stuff such as asking it fun facts, jokes, reminders, etc.
* I recently felt the need to change my home office setup. 
I've searched on Youtube for desk setups to get some ideas, and many had LEDs on it, which seemed to produce a really cool, chill effect (when not exagerated).

I've always been a fan of LED stripes, and even used them to display my lego corvette build (LINK TO TWITTER), and it creates a really cool effect. 
So I've decided to give it a try.

# Which LEDs should you buy?

!> As you might have understood by the title, this will be a DIY solution. If you don't want to build it yourself, there are some LED stripes that already have integration with smart assistants.

When I first bought some of this LED stripes, I didn't even knew that there are different types, such as the ws2808, ws2812B, etc. 
My only criteria was that they had to have RGB. 
I've ended up buying the ws2812B because it was the model to which I saw most tutorials.

Nowadays there's an even wider variety of such stripes. Is really easy to find stripes with a controller, with presets and even with integration with smart assistants such as Siri, Alexa, Google Assistant, etc.

But because I already had some ws2812B LEDs, I didn't want to spend more money just to have the integration with Google Assistant.

# Let's see how can we achieve it

After an initial research, I've found through [this tutorial](https://iotdesignpro.com/projects/google-assistant-controlled-led-using-ESP32-and-adafruit-io) that's possible to use IFTTT and Adafruit IO.
My main problem with this tutorial was that it uses the ESP32, and I wanted to use a Raspberry since it's what I had laying around.
With that said, here's what I ended up using:
* RaspberryPi - I've used a 3b because I was not able to make the LEDs work with the pi0.
* Ws2812B LED stripe - I've used one with only 29 LEDs (had one missing) because I don't to use an external power supply.
* IFTTT 
* Adafruit IO 
* A python script

## What's IFTTT and Adafruit IO and why are they required?

IFTTT stands for "IF This Than That" and it is essentially an easy way to automate tasks and allow apps to "communicate" between them. I see it as a bridge between apps.
For this scenario, it will be the bridge between the Google Assistant and Adafruit IO.


Adafruit IO ["is a platform designed (...) to display, respond, and interact with your project's data."](https://learn.adafruit.com/welcome-to-adafruit-io).
In this case it is used to store the keywords sent to the Assistant, such as "ON", "OFF", "RED", "GREEN", etc.

The way this works is that you define what you want to say to the Assistant and what are the keywords.
This way, when you talk to the Assistant, _IF_ what you said matches what you defined (_THIS_), _THEN_ the keyword you said will be sent to the Adafruit IO (_THAT_).

Here's a visual explanation:


For this part I've followed the guide I've mentioned, but I will outline here what you have to do so that you don't go back and forth between blogs.

1. Head over to the Adafruit IO [website](https://io.adafruit.com/) and create an account if you don't have one yet.
2. After doing so, head to the "Feeds" section and create a new Feed
![createFeed](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_create_feed.png)
3. Now create a new Dashboard
![createDashboard](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_create_dashboard.png)
4. Add a new button
![createButton](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_create_block.png)
5. Create a new toggle button and select the feed you've just created. Then press next step and you can leave the default, since we will not use it.
![createToggleButton](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_create_button_toggle.png)
6. Last but not least, you will need an API key. 
Treat this key like one of your passwords. 
Don't share this with anyone, or they will have access to your data.
![AIOKeyButton](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_key_button.png)
![AIOKey](/img/Make-your-ws28xx-LEDs-smarter-by-integrating-them-with-google-assistant/AIO_key.png)

Now let's move the the IFTTT part:

1. Head over to the IFTTT [website](https://ifttt.com/) and create an account if you don't have one yet.
2. Next create a new applet

## How is this all connected?

After setting up IFTTT and Adafruit, now you just need to connect the LEDs to the raspberry and write a script that will fetch the results and light up the LEDs.
For that I've based my script on the [Adafruit's examples]() for the Adafruit IO python library.
Then I used one of [their examples]() again on how to use the ws28xx python library.

Make sure you have the required libraries.
You can install them by doing:
* sudo pip3 install 

NOTE: As [stated by Adafruit](), `sudo` is required so that the library can interact with the hardware.

Here's a gist with the script that I wrote and an explanation:




