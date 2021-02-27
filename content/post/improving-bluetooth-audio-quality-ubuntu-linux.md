---
title: "Improving Bluetooth Audio Quality on Ubuntu Linux"
slug: "improving-bluetooth-audio-linux"
tags: ["Projects", "Linux"]
date: 2020-10-12T00:25:47+01:00
images: ["/bt-audio-cli.png"]
description: "Bluetooth Audio on Linux is a bit of a mess. There's a fair bit you can do to significantly improve the quality though."
draft: false
---

[Hacker News Discussion](https://news.ycombinator.com/item?id=24763593) | [Reddit Discussion](https://www.reddit.com/r/Ubuntu/duplicates/ja9kch/improving_bluetooth_audio_quality_on_ubuntu_linux/)

Bluetooth Audio is generally considered convenient, but not 'audiophile'. Any self respecting audiophile is probably connecting their fancy headphones to some Digital to Analog Converter/Amp combo which might be connected to their computer using some fancy gold plated USB Cable. More power to them, I am not self respecting. I made a decision a few months ago to switch entirely to Bluetooth Audio as the convenience and tidiness of having a wireless headset just appealed to me so much.

I purchased the M-Pow H21s - which are a fairly budget noise cancelling pair of Bluetooth headphones. They appealed to me because they weren't enormously expensive, and other M-Pow stuff I'd purchased in the past was of good enough quality for me to trust these would be too. I wasn't wrong. The noise cancelling is reasonably good, and the sound profile, while not 'neutral' - is totally fine for me to use while working.

One thing that really did annoy me about these headphones (and I'm not sure if the fault lies with Bluetooth Audio on Linux or with the headphones themselves) is that if I walked away from my desk and then returned, the audio quality would perceptibly drop, but never quite recover. It seemed to get 'stuck' at a lower quality, until the headset was rebooted. One day, I got tired of this, so decided to research what was going on.

# Codecs in General
In the beginning, there was A2DP - or the Advanced Audio Distribution Profile. This is a Bluetooth profile - which describes a method by which audio can be transmitted between a sender and a receiver. The profile mandates that all Bluetooth devices support a codec called SBC - or Low Complexity Subband Codec. A codec defines exactly how the device that is sending the audio should compress it. SBC is what's known as a Low Complexity codec. Low Complexity codecs have the design goal of being easy to encode for the sender and being easy to decode for the receiver. It's a fairly old codec, being the precursor to MP2 - itself the precursor of MP3. When consumers started demanding better quality audio out of their Bluetooth hardware, various other codecs were added to hardware. Some hardware added MP3 encoding and decoding - which was a small improvement in two ways. Firstly, if the source material was already MP3, and the encoder was smart enough, the MP3 data could directly be sent to the receiver, meaning that no encoding step was necessary. Secondly, MP3 is generally considered to be a better codec in terms of the resulting audio quality, so even if there was an encoding step, the results were generally better. Apple added AAC support to their hardware - which itself is a far superior codec to MP3 lends much the same benefits. The freshest codec which seems to be available is from a company that Qualcomm gobbled up - aptX. This codec is interesting in the sense that it only offers the second advantage talked about above. Nobody has music in aptX format, which meant that all source material must be re-encoded on the device sending the audio.

# A Better Codec, or a Higher Bitrate?
The first step to identifying if improvements can be made to your own Bluetooth Audio is to figure out which codecs your Bluetooth Headset supports. You can generally find this out by finding the manufacturers information. My particular headset, the h21 - is dumb as rocks, and only supports SBC. If your headset supports a better codec, especially if it supports a codec called LDAC - you should very much consider trying that before trying to do what we'll be doing in this article, which is improving the quality available to us using the SBC codec. [This article does a reasonably good job of covering all the codecs.](https://www.nextpit.com/bluetooth-audio-codecs)

No matter what your headphones support, the next thing you're going to want to do is install a custom PulseAudio module which both enables support for a bunch of more fancy codecs, and also allows you to configure the codecs. Go grab [EHfive/pulseaudio-modules-bt](https://github.com/EHfive/pulseaudio-modules-bt/wiki/Packages) and restart PulseAudio. You should already see in your systems sound settings you can now actually select between AAC, APTX, APTX HD, and LDAC - if your headset supports it. You can probably stop here if you're able to switch to APTX or LDAC. If not, it's time to configure SBC to be less sucky.

{{< figure src="/bt-audio-cli.png" title="default.pa, with the changes" >}}

So, a quick review of the options the SBC encoder has that we likely care about. Firstly, the Stereo mode. Generally in audio the two stereo modes we have are.. well Stereo and Joint Stereo. Stereo transmits two distinct channels of audio in the same stream, but completely distinct from one another. Joint stereo makes the assumption that the left and right channels are probably similar enough that just encoding how the right channel differs from the left will result in a more efficient packing of data. Your headset is likely already using one of these two modes. SBC actually supports a third mode, called dual - which essentially sends two audio streams to your headset, completely independently of one another. This effectively doubles the bit-rate your headset can operate at, since you now will have two separate streams of audio being encoded, rather than just one. If your headset supports this dual mode, and it most likely does, you definitely want it. In fact, this unofficial feature has been added to a popular custom Android OS - called LineageOS, [and they describe the benefit of this very well here](https://www.lineageos.org/engineering/Bluetooth-SBC-XQ/).

The next option we likely care about is the bitpool. The bitpool effectively determines the bitrate the audio will be encoded at. The higher, the better. [A calculator is available here](https://btcodecs.valdikss.org.ru/sbc-bitrate-calculator/) but what is shows is that at the default highest available Bitpool value (53) in Joint Stereo mode - the maximum available bitrate is 328kbps. If we do nothing else, and just switch to Dual Channel mode - the bitrate predictably roughly doubles to 617.4kbps. Some headsets, like my H21s, actually seem to do fine at higher bitpool values. Through experimentation, I found that around 70 was as much as my headset could manage without hearing the audio equivalent of a buffer overflow. At a bitpool value of 70, in dual channel mode, the bitrate now gets to 804.8kbps!

If you are experimenting with a different headset, I suggest starting with a bitpool value of 53 and dual stereo enabled. You can then experiment with increasing the bitpool value, as described below.

To modify these values, you'll need to edit ```/etc/pulse/default.pa```. Firstly, find the lind that begins ```load-module module-bluetooth-discover```. Modify it to add these flags, like so:

```load-module module-bluetooth-discover a2dp_config="sbc_cmode=dual sbc_min_bp=53 sbc_max_bp=53 sbc_freq=44k"```

This will lock your SBC encoder to work at a bitpool size of 53 - in dual stereo mode. Restart PulseAudio by doing ```pulseaudio -k``` - and reconnect your Bluetooth headset. Hopefully, you'll perceive a large improvement in quality, as I did. You can now experiment by changing that value of 53 for a higher value. [There's a list of known compatibility available here](https://btcodecs.valdikss.org.ru/codec-compatibility/).

The one downside of this approach of locking the encoder to work at a particular bitpool is that you lose graceful degredation of quality as the link quality reduces. You could experiment with having a lower value for the `sbc_min_bp` so that the encoder can reduce quality.

Let me know if this helps you!

[Hacker News Discussion](https://news.ycombinator.com/item?id=24763593) | [Reddit Discussion](https://www.reddit.com/r/Ubuntu/duplicates/ja9kch/improving_bluetooth_audio_quality_on_ubuntu_linux/)
