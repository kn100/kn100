+++
author = "Kevin Norman"
categories = ["Projects", "Pi", "Ghost"]
date = 2018-03-29T00:24:13Z
description = ""
draft = false
slug = "ghost-on-a-pi-because-why-not"
tags = ["Projects", "Pi", "Ghost"]
title = "Spooky Ghost on a Pi, Because Why Not?"

+++

This is a description of how this blog is currently hosted. Hopefully, I'd like to do more interesting stuff with the Pi and post about it here.

## Hardware
The system currently hosting this blog is a Raspberry Pi 3b+. This is the (at the current time) latest revision of the single board computer, and features a slightly higher clock rate over the 3b, as well as better network performance. This is attached to a pretty standard 2 amp capable wall wart.

## Networking
The Pi is connected to my home network (a 80mbit down 20mbit up VDSL service) via WiFi. This is mainly for convenience, and does limit my performance somewhat. Anecdotally speaking, I get about 30-35mbit/s of throughput either way. This is greater than my upload speed, so this is fine for me. Currently, the domain is pointing at DigitalOcean Name servers, which then in turn point at my local IP. Not ideal since that can change, but I'll sort out a DynDNS like service up later. My router then forwards specific ports through to the Pi.

## Software
I wanted something fairly simple, so I went with the following. I firstly formatted the MicroSD card of the Pi with Raspbian Stretch. The Pi is running NGINX, which handles SSL termination. The SSL certificate came from [Lets Encrypt](https://letsencrypt.org/). Nginx reverse proxies to Varnish, which is configured to use 20% of whatever RAM is available on the Pi, which at the current time would be around 170mb of ram for Varnish. Plenty, for a basic blog! The blog itself is running on the very nice (but somewhat fiddly to set up) Ghost. I'd recommend anyone who is attempting to set Ghost up on a Pi to not use their CLI. It's not that difficult to set it up without and the CLI obscured a bunch of random permissions issues I was not expecting. 

## Future plans
I want to host other stuff on the Pi, as well as fitting the Pi inside of my main computer. This is mainly to have a machine that remains on when the main machine is off. This is useful for a number of reasons. For me specifically, I look forward to having an IRC bouncer that I own running on the local machine, as well as configuring it to record some radio shows I sometimes want to listen to, but generally not at the time they air!
