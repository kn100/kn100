---
title: "Why Does My Computer Not Boot with a USB Hub Attached?"
slug: "usb-hub-bs"
tags: ["Projects", "Hardware"]
date: 2020-10-01T21:00:00+01:00
images: ["/hub-circuit.jpg"]
description: "USB Hubs are a minefield. They're cheaply designed, poorly thought out, and in some cases can potentially damage attached devices. This blog post explores why, and how to fix it."
draft: false
---

My trusty old USB 3 hub has failed. It was only around a year old, but it was cheap and it worked. It wasn't powered, but most of the time it didn't need to be. One day I began having very strange issues with USB on my computer. Devices randomly disconnecting, devices never connecting until the computer was rebooted and so on. I eventually narrowed it down to the hub. I tossed it out after a quick internal inspection had revealed nothing obviously wrong, and went shopping for a new hub.

My requirements for a hub were very simple:

* it must be externally powered,
* it must be relatively aesthetically neutral - no gamer aesthetics or hideous glossy plastic,
* it must be USB 3,
* it must offer between 3 and 7 extra ports - any less and what's the point, and more and the hub is unnecessarily big, and I treat my desk like Tokyo. Space is a at a premium,
* the cost must not exceed 35 GBP (45 US Dollars at the time of writing),
* and it must have a detachable cord that connects it to the computer, or offer a cord long enough to reach my computer under my desk.

While shopping, almost every single hub I came across had some stupid design that either prioritised it looking space age with blue LEDS galore, and a lot of them even had individual power switches for each USB port. Why anyone would want that is beyond me. Other hubs had tiny 15cm cables, since they're designed to be used right beside a laptop.

I ended up settling on a very cleverly designed Orico 4 port Aluminium hub. I particularly liked this one because aesthetically speaking it was pretty neutral, and it even handles the problem of the USB hub floating off in the breeze by integrating a clamp so that you can clamp it to your desk or your monitor.

{{< figure src="/hub-product.jpg" title="The hub I chose. A novel design!" >}}

It arrived, and seemed to work perfectly. USB transfer speeds were as expected, the external power source being MicroUSB meant I could attach any fairly dumb USB power supply to it to supply extra power, the cable leading to the PC itself was detachable and relatively standard (USB A to USB A) and all was good with the world.

That was until I put my computer to sleep. The next morning, I pressed a key on my keyboard to trigger the system to wake up, and nothing happened. The keyboard is not attached to the hub, nor is it attached to a port on the motherboard that is on the same controller, so this was perplexing. I pressed the power button on the tower, but still nothing! Very strange. I detached the USB hub, and immediately my computer came to life and worked as normal.

Through further experimentation I determined this strange phenomenon only happened when an external power source was connected. I toyed with the idea of returning what was obviously a faulty product, but reasoned that I didn't actually need it to be powered, and pretty much all the other hubs in the same price range were either aesthetically disgusting or lacked other features I liked about this one - so I kept it.

That is until today, when through some research, I found this isn't just a problem with *my* USB hub, it's a problem many have with powered hubs. Articles like [this](https://www.pro-tools-expert.com/production-expert-1/2019/9/18/warning-your-usb-hub-may-be-harming-your-drives-and-you-may-lose-valuable-studio-work-heres-how-to-fix-it), or forum posts like [this](https://forums.tomshardware.com/threads/computer-wont-start-with-usb-hub-connected.2753255/) show that totally unrelated hubs cause similar issues for their owners. This got me thinking to past experiences of USB hubs I've had. In the past, a less financially stable me used to buy the cheapest possible thing every time. At one time, I was experimenting with a Raspberry Pi along with external hard drives. I discovered that a powered hub was necessary for these experiments because the Pi itself was incapable of providing much USB power. I bought the cheapest USB 2 powered hub I could at the time from eBay, and it mostly worked fine. To describe the setup a little, I had the hub itself connected to its power supply, and the 'in' usb port connected to one of the Pis USB ports. Imagine my surprise when the Pi switched on and booted up, without any power supply attached! I thought this was cool at the time, and chalked it up to 'It's not a bug, it's a feature', and forgot about it. Fast forward to today me, when I realised my experience back then was possibly related to the problems I was having with this new Orico USB hub today. After a little bit of thought, I realised that the hub was most likely 'backfeeding' power to the connected computer.

# What is backfeeding?
To keep things simple, let's stick to USB 2 for now. USB 2 consists of 4 conductors, known as VCC (this is the 'positive' 5 volt connection), GND (this is the 'negative' power connection), D+, and D- (the data transmission pins, no true power flows over these pins). This means that a connected USB device can both draw power and transfer data. When you see 5 volts on a power adaptor or something, this is usually an approximation. For various reasons which revolve mostly around cost saving, most USB power adaptors will actually output slightly more than 5 volts, and then when a load is applied (say, a charging phone), the voltage drops down a bit. The better the quality of the power supply, closer to 5 volts it'll start at generally and the smaller the drop when a load is applied.

{{< figure src="/usb-pinout.png" title="The USB pinout" >}}

This applies to your computer's USB ports too - except for one big difference: the power supply inside your computer is probably significantly better designed than the USB plug sockets that are mass produced. In fact, if I measure the voltage coming from my USB ports on my computer, I get a value of around 5.03v. If I measure the voltage coming from my phone charger, I get 5.21v. This is a pretty big difference!

The next thing to know is how a powered hub actually works. Essentially, a powered hub takes an external power source, and passes that power source through to the USB devices connected to the hub in place of your computer. One problem that these hubs face however is what if the user wishes to use the hub in its 'unpowered' state - where they've connected the hub to their computer with no power supply. In order to implement this properly, the hub would probably need some mechanism to switch between the computer power and the external PSU if it is connected. This would however add cost to the manufacture of the hub, so instead of doing this, they just connect the 5v of your computer through to the 5v of the external power supply. This is ridiculously bad because if there is even the smallest difference between the voltage being supplied by your computer and the voltage being supplied by the external power source, power will flow towards the device with the lower voltage! This is 'backfeeding'. As a simple analogy, imagine you took two rechargeable batteries and connected them together. What you'd find happens is that the batteries will eventually end up at exactly the same voltage, which will be slightly lower than what you'd expect if you did the maths, as a small amount of power would be lost as heat. If the power source is 'infinite' however - like our wall connected computer or our wall connected external hub, this power will just continue to flow. Where it goes or what happens with it is completely undefined. In the case of our Raspberry Pi earlier, the circuit was simple enough that it just led to the Pi being powered up. In the case of my computer, it screwed with the system so badly that hardware buttons like the power button literally stopped functioning altogether.

# Why would they design it like this

This design seems stupid right? The problem is, it is the cheapest way of achieving the design goal of having a USB hub work with or without power. If we didn't care about this particular feature, we could either just not have an external power source and have a purely unpowered hub, or we could not connect the 5v pin from the computer and therefore only supply power from the external PSU. Both of these solutions in effect are more expensive than just living with the backfeeding 'feature' since it'd mean the company offering the hub would need to deal with support requests from people wondering why their hub doesn't work 'unpowered' when their other one does, or vice versa. Instead, we get backfeeding. I'm sure that some computers would not exhibit any real problems with this design, seeing as it's been around since as long as powered USB hubs have been around as far as I can tell, and it's likely some USB chipsets are designed with this in mind, but mine was not. Others computers, especially Macs, deal with this particular issue far more destructively, causing damage to the computer's USB chipset when a backfeeding USB hub is connected.

# Disclaimer

Before we continue, you should know I am not an electrical engineer, if that wasn't clear from how vaguely I described things above. I am a software engineer who likes to dabble with electronics. This means my advice below does not come from someone who is qualified to give it. You should follow this advice at your own risk. I take no responsibility for damage you do to yourself, your USB hub, or your computer, or to anything else for that matter.

# Fixing this BS for ourselves

1. Firstly, I validated this was the problem. I engaged in a clever little trick where I took an old USB cable - USB 2 or 3 is fine, cut it in half, exposed the either 2 (cable is power only) or 4 wires, and looked up USB wire colouring to realise that the red wire is usually the VCC wire, and Black/White is usually GND. I then connected the cut up USB lead to the input port on my hub (where my computer connects), and connected the external power supply. I measured 5 volts across these two wires using the multimeter, meaning this hub does backfeed.

{{< figure src="/hub-screw.jpg" title="The hubs screws" >}}

2. My hub comes apart very easily. Disconnect all wires. Four screws removed and we immediately get access to the circuit board. USB 3 is a little more complicated than USB 2 - featuring 9 conductors rather than USB 2s 4.

{{< figure src="/hub-circuit.jpg" title="The hubs circuit board. Surprisingly nice, given this stupid issue!" >}}

3. I have no idea what the pinout of USB 3 is, nor how this specific connector orders things.  I connected the cut up USB lead to the input port on my hub (where my computer connects). We can then attach our multimeter in continuity mode to the red wire, and used the other probe to probe about the USB port to figure out which of the pins is VCC. In my case, it was the 8th pin from the left.

{{< figure src="/hub-measure.jpg" title="Identifying which pin carries 5 volts" >}}

4. I snipped this pin using my wifes beauty scissors. If you do this same thing, I recommend not telling your wife.

{{< figure src="/hub-cut-zoom.jpg" title="Cutting this pin, to break the connection." >}}

5. I reassembled the hub, and connected it only to power. I connected my multimeter in voltage measuring mode across both wires of our cut up USB lead. I measured zero volts. This is to validate the work we have just done. Our hub no longer backfeeds!
