---
layout: post
title: The Teracube 2e
subtitle: The story of the Teracube 2e
tags: [tech, android]
---
The Teracube 2e is a budget smartphone with a 4 year warranty from [Teracube](https://myteracube.com), a startup focused at eco-friendly smartphones having a really long life.
<img style="float: right" src="https://raw.githubusercontent.com/gaganmalvi/graphics/main/photo_2021-04-07_15-52-20.jpg" height="320"> 

Specs wise, it has a Mediatek Helio G25 (MT6762), with 4 GB of RAM, and 64GB of onboard storage. It comes with a stock Android experience right from the factory. 

The company promises 3 years of Android updates.

I received my Teracube 2e in April 2021, through a developer program they had held, they had taken the pains to get it shipped to India as soon as possible. I would like to thank Sharad Mittal, the founder of Teracube, and my mentor and my friend Kshitij Gupta (AgentFabulous) for giving me such an amazing opportunity.

The device in hand, even with a plastic back, felt premium for a Redmi user like me, and it actually felt like a sturdy phone, unlike some devices in the 2021 era.

The case bundled with it provides an extra layer of protection, and the best part is, it's made out of fully recycled materials and feels really good in hand.

<img style="float: left" src="https://raw.githubusercontent.com/gaganmalvi/graphics/main/photo_2021-04-09_10-56-27.jpg" height="320"> 

Soon after receiving it, I unlocked the device, and started working on getting the highly-popular custom Android-based aftermarket firmware or custom ROM, named [LineageOS](https://lineageos.org). It is the successor of the highly-popular defunct ROM, CyanogenMod.

Teracube was also kind enough to release the source for TWRP, which is now officially supported from the company for the device, hence opening new avenues for the modding and development of the device.

I am also very thankful to Kshitij Gupta for holding my hand throughout this development journey, teaching me and guiding me without any hesitation so I could be capable of bringing aftermarket firmware to the device.

I started off by making a dump of the stock firmware, using [ShivamKumarJha's scripts.](https://github.com/ShivamKumarJha/android_tools) Using these scripts again, I generated a dummy vendor and device configuration for the Teracube 2e. More about the configurations in such trees can be read [here.](https://source.android.com/setup/develop/new-device)

After that, I started adding necessary configuration arguments needed to actually compile the operating system for the device, i.e. the changes required to the [fstab](https://en.wikipedia.org/wiki/Fstab), and different packages for audio, display, media, bluetooth, NFC, and so on. 

I had referred to present MediaTek device repositories of the following devices:

 - Redmi Note 8 Pro - maintained by Kshitij Gupta
 - Redmi Note 9T - maintained by Vaisakh Murali
 - Realme C2 - maintained by Sakthivel Nadar
 - Gigaset GS290 - maintained by Erfan Abdi

Now, since the kernel source for the Teracube 2e was not available at that moment, so I proceeded to compile using a prebuilt kernel binary extracted from the stock firmware.

For compiling, we'd also need the kernel headers for the prebuilt kernel, for that dependency I took a leaked MediaTek 4.9 kernel base and took the headers from there.

After the compile process, I had my first brick.
Why?
I messed up the initialization process. 

MediaTek in their device board support packages, tend to use properties instead of hardcoded paths in their init.*.rc files, and since I hadn't defined the property, services just failed to initialize, throwing the device into fastboot. This was fixed with [this commit.](https://github.com/mvaisakh/android_device_xiaomi_cannon/commit/b1ace13d3e0d24816c372b72c42b75e650c1c39c)

Soon after fixing this, I was missing a couple blobs and configuration XML files that caused the device not to boot, but to stay on bootanimation, what we term as a bootloop. One of the errors that I had faced, was 
```
04-08 12:06:06.683  1020  1020 I FastMixerState: sMaxFastTracks = 8
04-08 12:06:06.685  1020  1020 V MediaUtils: physMem: 4030423040
04-08 12:06:06.685  1020  1020 V MediaUtils: requested limit: 536870912
04-08 12:06:06.685  1020  1020 I libc    : malloc_limit: Allocation limit enabled, max size 536870912 bytes
04-08 12:06:06.686  1020  1020 I audioserver: ServiceManager: 0x7d66bbadc0
04-08 12:06:06.688  1020  1020 I audioserver: type=1400 audit(0.0:4675): avc: denied { call } for scontext=u:r:audioserver:s0 tcontext=u:r:init:s0 tclass=binder permissive=1[
04-08 12:06:06.696  1020  1020 I AudioFlinger: Using default 3000 mSec as standby time.
04-08 12:06:06.697  1020  1020 E APM::Serializer: deserialize: Could not parse /odm/etc/audio_policy_configuration.xml document.
04-08 12:06:06.697  1020  1020 E APM::Serializer: deserialize: Could not parse /vendor/etc/audio/audio_policy_configuration.xml document.
04-08 12:06:06.704  1020  1020 E APM::Serializer: deserialize: libxml failed to resolve XIncludes on /vendor/etc/audio_policy_configuration.xml document.
04-08 12:06:06.708  1020  1020 E APM::AudioPolicyEngine/Config: parse: Could not parse document /vendor/etc/audio_policy_engine_configuration.xml
04-08 12:06:06.708  1020  1020 W APM::AudioPolicyEngine/Base: loadAudioPolicyEngineConfig: No configuration found, using default matching phone experience.
04-08 12:06:06.708  1020  1020 E APM::AudioPolicyEngine/Config: parseLegacyVolumeFile: Could not parse document /odm/etc/audio_policy_configuration.xml
04-08 12:06:06.714  1020  1020 E APM::AudioPolicyEngine/Config: parseLegacyVolumeFile: libxml failed to resolve XIncludes on document /vendor/etc/audio_policy_configuration.xml
04-08 12:06:06.714  1020  1020 E APM::AudioPolicyEngine/Config: parseLegacyVolumeFile: Could not parse document /system/etc/audio_policy_configuration.xml
04-08 12:06:06.714  1020  1020 E APM::AudioPolicyEngine/Base: skipped 1 elements
```
```aurisys``` configuration XMLs, and because of a missing ```default_volume_tables``` XML that I needed to copy from ```frameworks/native```, caused it to bootloop.  Soon after fixing the ```vtservice``` error and the missing XML issues, the device booted, and fixing the remaining bugs on the device was a cakewalk, since most of the issues either were easy fixes, and the rest were already documented.

<img src="https://camo.githubusercontent.com/b8e2ed720715e011a68cfe244217da44da6adeaa303ec75e11a30ef9f475cbad/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4579663948453456494149673845783f666f726d61743d6a7067266e616d653d6d656469756d" height="480">

Soon after LineageOS, I also booted [/e/OS](https://e.foundation), a privacy focused Android distribution, that also comes preinstalled on many Android devices. 

There sure are a few bugs left, namely:

 - WiFi calling
 - Bluetooth Audio

I hope to fix them before I move forward to bring Android 11 to the Teracube 2e :) 

The device sources for the Teracube 2e shall be opensourced soon [on to the Teracube 2e device organization](https://github.com/2e-dev).

I would like to thank the following people for bringing this to life:

 - Sharad Mittal, founder of Teracube
 - Kshitij Gupta, my mentor, and a great friend
 - Sakthivel Nadar
 - Elias
 - Vaisakh Murali
 - Zinadin Zidan
 - Sahil Sonar
 - And whomsoever I may have missed

Thank you for reading my story :)
