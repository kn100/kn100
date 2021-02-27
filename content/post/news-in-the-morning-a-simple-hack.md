+++
author = "Kevin Norman"
categories = ["Projects", "hardware"]
date = 2018-06-03T19:43:56Z
description = ""
draft = false
image = "/images/newshack.jpg"
slug = "news-in-the-morning-a-simple-hack"
tags = ["Projects", "Hardware"]
title = "News in the Morning: A Simple Hack"

+++

Like many people, I find it difficult to wake up in the morning. I want to automate some of the things I do in the morning in order to continue to hack away at my laziness. 

To this effect, I want my television to switch on and over to BBC news in the morning. Unfortunately, Hisense do not provide a way to control the television remotely as far as I can tell, and `nmapping` it doesn't give me anything too interesting. The only way I can think of to interact with the thing is via Infrared, so that's what I'll do.

So far, I've acquired the following hardware:
* China-Arduino Nano
* KY-022 Infrared IR Sensor Receiver Module
* KY-005 IR LED Module Transmitter

I hooked this all together in a sensible fashion.

{{< figure src="/images/newshack.jpg" title="The circuit I built" >}}

Next up was to figure out how the heck IR transmission worked. It turned out to be a little more complicated than I had imagined. It turns out there's a bunch of different standards for data transmission (surprise surprise..), from a bunch of different manufacturers. As usual, nothing is quite that simple and my television was not listed as a supported standard with the [library I was using](https://github.com/z3t0/Arduino-IRremote) which is why I purchased the receiver. 

Hopefully, I could 'learn' codes from the remote control and rebroadcast them at will using the transmitting IR led. 

A preliminary test using the [IRecord example](https://github.com/z3t0/Arduino-IRremote/blob/master/examples/IRrecord/IRrecord.ino) showed this theory to mostly work! It turns out that this particular television (the Hisense H43N5300) makes use of a standard called RC6, developed by Philips.

The next steps for me will be to learn the button sequence required to turn the television on, flick to the news, and potentially slowly increase the volume until I disable it somehow. After that will come the fun process of disassembling a cheap alarm clock and fitting it all inside there, ready to be triggered by my alarm.
