+++
title = "Gnome Shell Stuttering Caused by AppIndicator"
date = 2020-05-19T17:51:48+01:00
draft = false
slug = "gnome-shell-stuttering-caused-by-appindicator"
tags = ["linux"]
+++

I recently installed Pop!_OS on my new desktop machine and have been loving it, but have been suffering with this really strange issue where the entire UI would stutter roughly every second for a few milliseconds. This got really, really annoying so I had a little dig around and found out the cause was actually a strange interaction between Ubuntu AppIndicator (a bundled extension for Gnome Shell in Pop!_OS) and YTMDesktop - an electron wrapper for Youtube Music.

I installed [YTMDesktop](https://ytmdesktop.app/) using Snap - a tool I don't fully understand nor like very much. I installed it using Snap since as far as I can tell it's a nice way to install apps you don't care that much about and supposedly provides some security niceties too. After I figured out that the stuttering only happened while Youtube Music was playing music, I erroneously reported the bug to the Gnome Shell team. I even attempted to record the issue using a screen recording tool, but the stutters were bad enough that the screen recording tool itself was stuttering too and therefore not picking up on the lost frames! Almost immediately someone there pointed out it might be related to AppIndicator - so I tried disabling it through the inbuilt extensions menu in Pop!_OS (but you can install the Gnome Tweak Tool if this isn't available for you!) the stuttering immediately stopped! You can see this for yourself if you're affected by using something like the [Blur Busters UFO test](https://www.testufo.com/framerates#count=1&background=stars&pps=960).

I dug a little deeper and found in my `journalctl` there were hundreds of entries from YTMDesktop where it was attempting to access some resource in a flatpak related directory!

```
May 18 13:39:15 pop-os audit[36663]: AVC apparmor="DENIED" operation="open" profile="snap.youtube-music-desktop-app.youtube-music-desktop-app" name="/home/kn100/.local/share/flatpak/exports/share/icons/hicolor/48x48/apps/" pid=36663 comm="youtube-music-d" requested_mask="r" denied_mask="r" fsuid=1000 ouid=1000
```

It seems like maybe (and I do not understand Snap or Flatpak at all - although both are in use on my system) the Youtube Music Desktop App was originally packaged for Flatpak and then stuffed into a Snap. It's trying to access nonexistent resources now which assumedly is why AppIndicator is crashing.

It seems absolutely crazy to me that an extension to Gnome Shell could cause such a UI disaster - and wonder how many other apps cause similar issues. Users everywhere might install one of these apps and decide that Gnome Shell is garbage, as I have a few times in the past. From a little Googling, it seems that Discord and Megasync are both also affected. Discord seems a little egregrious given that it's mainly used while gaming - an activity where stuttering is unacceptable.

For now I have disabled AppIndicator - but now certain applications I use have significant usability issues. When you close them, they remain open in the 'system tray' that no longer exists, and therefore you have no way to access them now!

I understand the Gnome teams design goal of getting rid of system icons, but given that basically every OS packages AppIndicator or something similar since some apps absolutely require you to have access to their tray icon - it seems like maybe this design goal has hit the rails of reality and needs to be reconsidered.

I've added my issue to the already existing issue on Github [here](https://github.com/ubuntu/gnome-shell-extension-appindicator/issues/226).