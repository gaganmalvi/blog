---
layout: post
title: My Android 11 Bringup Story
subtitle: Let's get Android 11 working on the Teracube!
tags: [tech, android]
---

For a bit of a backstory on my Teracube 2e journey, you can read it over [here](https://malvi.ml/blog/2021-04-22-the-teracube-2e/)

Now, we got Android 10 stable and working. Literally quashed every bug except VoWiFi. Help is welcome :P

Here comes the elephant into the room - we have Android 10. But Android 11 is now mainstream.
What are we going to do?

Let's port Android 10 stuff over to 11 :)

First things first, I noticed both stock firmware and the custom firmware had bad rounded corner values. Ended up fixing them as soon as possible :)

I then dropped the Android Exception Engine, designed by Mediatek, that is an exception catching and debugging information generation mechanism. That anyways is kind of useless on custom ROMs, since we have way more methods to debug.

Since Android now uses a versioned naming approach for protobuf, I added that to the public.libraries.txt and also compiled the versioned one [as seen in this commit by ix5](https://github.com/2e-dev/android_device_teracube_2e/commit/23cc5156e45c441e2263b8af6172d73f2cad17e9).

Then, I enabled the version 1.3 of DRM plugins like ClearKey.

I also built libhwbinder and libhidltransport (both system and vendor variants) to ensure compatibility for the older blobs based on Android 10.

I also then moved seccomp policies to another repository containing stuff common across all MediaTek chipsets.

Alongside all this, in order to comply with Android 11 dynamic partition rules, I also initialized another new logical system_ext partition. The commit can be found [here](https://github.com/2e-dev/android_device_teracube_2e/commit/964e4bd62b0facacf58f2553d25bb74682401005).

I then loaded VNDK 29 versions of some blobs required for audio, and gatekeeper, and patchelf'd them accordingly to load whenever the parent blobs were loaded.

VSYNC BoardConfig flags were deprecated, so they were converted to properties later on.

There was vtservice, that needed a shim: 

_ZN7android10AudioTrackC1E19audio_stream_type_tj14audio_format_tjj20audio_output_flags_tPFviPvS4_ES4_i15audio_session_tNS0_13transfer_typeEPK20audio_offload_info_tjiPK18audio_attributes_tbfi

_ZN7android12rotateBufferEPhS0_PNS_10RotateInfoEPi

This mangled code turned out to be null functions, so we shimmed it as follows:

```
extern "C" {
   void _ZN7android10AudioTrackC1E19audio_stream_type_tj14audio_format_tjj20audio_output_flags_tPFviPvS4_ES4_i15audio_session_tNS0_13transfer_typeEPK20audio_offload_info_tjiPK18audio_attributes_tbfi() {}
   void _ZN7android12rotateBufferEPhS0_PNS_10RotateInfoEPi() {}
}
```

and the resulting blob was patchelf'd to load libshim_vtservice on init.

Now, here comes a hard thing - the older IMS stack wasn't compatible with Android 11.
Why?
CellBroadcastReceiver in the AOSP IMS stack, had been moved to a module, and hence, we couldn't shim it, or do something for it to work.

Once the boot JARs were enforced, booting was impossible since the MtkCellBroadcastReceiver class in the mediatek-telephony-common.jar asked for another AOSP accompaniment class which was not present, and hence, zygote check failed, and the device failed to boot.

I tried to remove the class from source, but that utterly failed too!

This was fixed by using OSS-IMS from PixelExperience (thanks Zidan!) with a newer revision of the mediatek-telephony-common.jar from an R MT6768 dump, namely "xiaomi-merlin" (thanks ashblk and surblazer!).

Along with OSS IMS, we also moved to some more source built components that Zidan had hacked to compile on AOSP, outside the BSP tree, for libladder and libudf, etc.

With everything booting, we then moved to Health HAL 2.1 and proceeded to clean up the device tree. 

During the bringup, ADB was dead, so we used a hack to store logcat into the /cache/boot_log.txt directory. The hack can be found [here](https://github.com/2e-dev/android_device_teracube_2e/commit/7954be203632011cdf55e333e1a0379cfcc9d3ce)

IMS also required a deprecated class to work - the ImsServiceBase class that was removed from frameworks/opt/net/ims.

With help from Kshitij Gupta at Teracube, we successfully created a boot JAR that had all the contents of ImsServiceBase, and was accessible globally throughout system. And hence that got shimmed as well :)

Ironing out debugging from the kernel and cleaning some stuff up, I then moved onto making the tree boot on SELinux enforcing. 

I then made device-specific SE policy compile on Android 11 MediaTek SE repository found on the PixelExperience GitHub, and was able to boot enforcing without many issues within a few hours. Soon enough, the device was stable as a rock.

Also again, VoWiFi is still being a b*tch.

Redmi Note 8 Pro was a beautiful reference throughout my bringup journey. 

After all this, I then started shipping SW6 firmware alongside my ROM - since there were some undocumented issues with custom ROMs running on it (and yes I was lazy, I admit). 

All the source code is on [2e-dev](https://github.com/2e-dev) and [AOSPA-2e](https://github.com/AOSPA-2e)

Sincere thanks to Kshitij Gupta, Erfan Abdi, Elias, Zidan and many other devs who helped me out throughout all this :)

Alright, sayonara till next time! :D