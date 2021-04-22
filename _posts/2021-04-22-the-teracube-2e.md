---
layout: post
title: The Teracube 2e
subtitle: The story of the Teracube 2e
tags: [tech, android]
---
![alt text](https://github.com/gaganmalvi/graphics/raw/main/teracube2e.png "The Teracube 2e.")

The Teracube 2e is a budget smartphone with a 4 year warranty from [Teracube](https://myteracube.com), a startup focused at eco-friendly smartphones having a really long life.

Specs wise, it has a Mediatek Helio G25 (MT6762), with 4 GB of RAM, and 64GB of onboard storage. It comes with a stock Android experience right from the factory. 

The company promises 3 years of Android updates.

I received my Teracube 2e in April 2021, through a developer program they had held, they had taken the pains to get it shipped to India as soon as possible. I would like to thank Sharad Mittal, the founder of Teracube, and my mentor and my friend Kshitij Gupta (AgentFabulous) for giving me such an amazing opportunity.

![Teracube 2e](https://raw.githubusercontent.com/gaganmalvi/graphics/main/photo_2021-04-07_15-52-20.jpg) ![Teracube 2e](https://raw.githubusercontent.com/gaganmalvi/graphics/main/photo_2021-04-09_10-56-27.jpg)

The device in hand, even with a plastic back, felt premium for a Redmi user like me, and it actually felt like a sturdy phone, unlike some devices in the 2021 era.

The case bundled with it provides an extra layer of protection, and the best part is, it's made out of fully recycled materials and feels really good in hand.

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
04-08 09:56:21.635  1028  1028 F linker  : CANNOT LINK EXECUTABLE "/system/bin/vtservice": library "vendor.mediatek.hardware.videotelephony@1.0.so" not found
04-08 09:56:21.732   962   980 W ContextImpl: Missing ActivityManager; assuming 1041 does not hold android.permission.UPDATE_DEVICE_STATS
04-08 09:56:21.760  1020  1020 F libc    : Fatal signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0 in tid 1020 (audioserver), pid 1020 (audioserver)
04-08 09:56:21.798  1038  1038 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
04-08 09:56:21.798  1038  1038 F DEBUG   : LineageOS Version: '17.1-20210408-UNOFFICIAL-2e'
04-08 09:56:21.798  1038  1038 F DEBUG   : Build fingerprint: 'Teracube/Teracube_2e/Teracube_2e:10/QP1A.190711.020/202011161116:user/release-keys'
04-08 09:56:21.798  1038  1038 F DEBUG   : Revision: '0'
04-08 09:56:21.798  1038  1038 F DEBUG   : ABI: 'arm64'
04-08 09:56:21.799  1038  1038 F DEBUG   : Timestamp: 2021-04-08 09:56:21+0000
04-08 09:56:21.799  1038  1038 F DEBUG   : pid: 1020, tid: 1020, name: audioserver  >>> /system/bin/audioserver <<<
04-08 09:56:21.799  1038  1038 F DEBUG   : uid: 1041
04-08 09:56:21.799  1038  1038 F DEBUG   : signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0
04-08 09:56:21.799  1038  1038 F DEBUG   : Cause: null pointer dereference
04-08 09:56:21.799  1038  1038 F DEBUG   :     x0  0000007c95618880  x1  0000000000000000  x2  0000000000000018  x3  0000000000000018
04-08 09:56:21.799  1038  1038 F DEBUG   :     x4  0000000020000000  x5  0000000000000018  x6  fefeff7aff647260  x7  7f7f7f7f7f7f7f7f
04-08 09:56:21.799  1038  1038 F DEBUG   :     x8  0000007c91ffb030  x9  00000000f0000000  x10 0000000000000001  x11 0000000000000000
04-08 09:56:21.799  1038  1038 F DEBUG   :     x12 0000000000000018  x13 ffffffffffffffff  x14 0000000000000004  x15 ffffffffffffffff
04-08 09:56:21.799  1038  1038 F DEBUG   :     x16 0000007c91ff9a60  x17 0000007c938db5cc  x18 0000007c9664c000  x19 0000007c91ffb068
04-08 09:56:21.799  1038  1038 F DEBUG   :     x20 0000007c95634800  x21 0000007c91ff79d8  x22 0000007fdc5c6490  x23 0000000000000000
04-08 09:56:21.799  1038  1038 F DEBUG   :     x24 0000007c91ffb030  x25 0000007c91ff7a08  x26 0000007c91ffb048  x27 0000007c91ffb068
04-08 09:56:21.799  1038  1038 F DEBUG   :     x28 0000007c95629f00  x29 0000007fdc5c6600
04-08 09:56:21.799  1038  1038 F DEBUG   :     sp  0000007fdc5c6430  lr  0000007c91fb228c  pc  0000007c91fb2434
04-08 09:56:21.807  1038  1038 F DEBUG   : 
04-08 09:56:21.807  1038  1038 F DEBUG   : backtrace:
04-08 09:56:21.808  1038  1038 F DEBUG   :       #00 pc 0000000000026434  /system/lib64/libaudiopolicyenginedefault.so (android::audio_policy::EngineBase::loadAudioPolicyEngineConfig()+2352) (BuildId: 785940a16ae0196921fea099a9d52664)
04-08 09:56:21.808  1038  1038 F DEBUG   :       #01 pc 000000000001d060  /system/lib64/libaudiopolicyenginedefault.so (android::audio_policy::Engine::Engine()+152) (BuildId: 785940a16ae0196921fea099a9d52664)
04-08 09:56:21.808  1038  1038 F DEBUG   :       #02 pc 0000000000024644  /system/lib64/libaudiopolicyenginedefault.so (android::AudioPolicyManagerInterface* android::audio_policy::EngineInstance::queryInterface<android::AudioPolicyManagerInterface>() const+144) (BuildId: 785940a16ae0196921fea099a9d52664)
04-08 09:56:21.808  1038  1038 F DEBUG   :       #03 pc 0000000000074000  /system/lib64/libaudiopolicymanagerdefault.so (android::AudioPolicyManager::initialize()+88) (BuildId: 3e0ba592c4ef91149260d93f24609aa5)
04-08 09:56:21.808  1038  1038 F DEBUG   :       #04 pc 0000000000012e70  /system/lib64/libaudiopolicymanager.so (android::AudioPolicyManagerCustom::AudioPolicyManagerCustom(android::AudioPolicyClientInterface*)+16) (BuildId: 19823d7cc571a21c73cc34f89b88c0a2)
04-08 09:56:21.808  1038  1038 F DEBUG   :       #05 pc 000000000000c168  /system/lib64/libaudiopolicymanager.so (createAudioPolicyManager+32) (BuildId: 19823d7cc571a21c73cc34f89b88c0a2)
```
```aurisys``` configuration XMLs, and because of a missing ```default_volume_tables``` XML that I needed to copy from ```frameworks/native```, caused it to bootloop.  Soon after fixing the ```vtservice``` error and the missing XML issues, the device booted, and fixing the remaining bugs on the device was a cakewalk, since most of the issues either were easy fixes, and the rest were already documented.

![LineageOS on the Teracube 2e.](https://pbs.twimg.com/media/Eyf9HE4VIAIg8Ex?format=jpg&name=medium)

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
