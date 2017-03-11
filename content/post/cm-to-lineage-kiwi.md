+++
Categories = [ "lineageos", "cyanogenmod", "android" ]
Tags = [ "lineageos", "cyanogenmod", "android", "huawei honor 5x", "adb", "twrp" ]
Description = "Transitioning from CyanogenMod to LineageOS on a Huawei Honor 5x"
date = "2017-03-11T15:58:08-05:00"
title = "From CyanogenMod to LineageOS on an Honor 5x"

+++

In case you hadn't heard, the Android operating system
[CyanogenMod is no longer around](http://lifehacker.com/cyanogenmod-is-dead-and-its-successor-is-lineage-os-1790554964).
I had been using CyanogenMod on both my phones (Nexus 5 and Honor 5X)
with great success: I'm a big fan of the project, even though
[at least one important person would contend it's less secure](https://github.com/WhisperSystems/Signal-Android/issues/127#issuecomment-13447074)
than stock Android, at least for the plebs. Like your parents.

Personally, I've chosen to flash CyanogenMod on my devices to avoid
some of the bloatware that comes packaged with Android phones upon purchase.
Not only that, but the Honor 5X must have been shipped with some
absolutely horrible software choices, because it was very sluggish,
even with at-the-time impressive hardware specs.

I decided I'd take a crack at "upgrading" to LineageOS 14.1 from
CyanogenMod 13.1 on the Honor 5X, mostly because I was having trouble
maintaining a WiFi connection to a 2.4 GHz network, which
[may have been resolved in a nightly update](https://review.lineageos.org/#/c/163842/).
I also figured that I'd have to update eventually, given the
absence of future CyanogenMod updates.

These notes are here to help others who may undertake the same task.
For those following along, I'm using Arch Linux on my laptop, so
commands in terminal form represent things that were run on Linux.

## TLDR

* Find your phone [here](http://wiki.lineageos.org/devices.html)
* If yours is also Honor 5X: keep reading. If not: hope the docs are correct
  and follow them.

### Kiwi users

0. Download the [build you want](https://download.lineageos.org/kiwi)
1. If you want Google Play Store / Android goodies,
   download [OpenGApps](http://wiki.lineageos.org/gapps.html).
   Be sure to read what each version offers before choosing.
   Kiwi uses `arm64` so be sure to pick that.
2. Get `adb` [installed and set up][adb]
2. If needed: enable developer mode on phone:
   `Settings`, `About`, `Build version` x 7 taps
2. Enable USB debugging, advanced restart options in developer options
2. Plug in phone via USB to machine with `adb` on it
3. Run these commands, noting possible file name differences:
    1. `adb push lineage-14.1-20170308-nightly-kiwi-signed.zip /sdcard/`
    2. `adb push open_gapps-arm64-7.1-pico-20170311.zip /sdcard/`
4. Reboot phone into recovery: hold power button, restart,
   look for Recovery Mode in options
5. Once `TWRP` loads, go to Wipe, then **individually**
   check each option, and swipe to wipe. E.g., Dalvik then swipe,
   cache then swipe, ... until finished.
   **WARNING**: you may want to save the contents of your microSD card,
   so don't wipe that if you do!
6. Still in `TWRP`, click install, queue up the LineageOS zip,
   then if you downloaded GApps, the GApps zip.
7. Flash
8. Reboot
9. Win

For those interested in the journey, read on.

## Downloading the appropriate files

It was easy to find the right files on
the [LineageOS site](http://lineageos.org/). I'm
using the kiwi device, which
[has a nice guide](http://wiki.lineageos.org/kiwi_install.html)
that walks a user through installation instructions.

I had to download two files: the
[update itself](https://mirrorbits.lineageos.org/full/kiwi/20170308/lineage-14.1-20170308-nightly-kiwi-signed.zip),
and [Open GApps](http://opengapps.org/?api=7.1&variant=pico).

I chose the `pico` version because I do want the app store, but I have
no interest in using "OK Google" nor unlock via face.

I then used `adb` to push these files to my phone's `/sdcard/` folder:

``` shell
adb push lineage-14.1-20170308-nightly-kiwi-signed.zip /sdcard/
adb push open_gapps-arm64-7.1-pico-20170311.zip /sdcard/
```

For those without `adb` set up but are running linux, the arch wiki
[has a nice guide][adb].
Those using other OSes may appreciate the info but need other software.

## Backing up

The first thing I did was back up all my data to my laptop.
I did this the "old fashioned way", mounting the device
and simply `rsync`ing files to my laptop's hard disk.
I used `simple-mtpfs` from [AUR](https://aur.archlinux.org/packages/simple-mtpfs/).
All I had to do was enable file transfer mode in the USB options
on my phone.

I then thought it'd be a good idea to take a more wholesome
backup, perhaps done via the phone itself. That's where I hit
the first of many issues.

## Device encryption

As someone concerned about my privacy, I've chosen to encrypt my
device. If you have an Android phone and are not sure if your device
is encrypted, you may want to
[get educated about it](https://www.howtogeek.com/141953/how-to-encrypt-your-android-phone-and-why-you-might-want-to/).
In the part of the world I'm currently in, phone theft is not uncommon
so it's important the data on my device cannot be easily accessed by
potential thieves.

In CyanogenMod, it was very easy to encrypt the device. Unfortunately,
there seems to be no way to decrypt the device. Adding more annoying
issues, the [recovery mod I'm using](http://teamw.in/project/twrp2/)
doesn't even prompt for an option to decrypt the device, and produces
several errors when attempting to access the encrypted partitions; this
makes sense as it can't access them.

That meant no possible way to back up the device without a micro SD
card (which somehow I decided against purchasing before leaving the US
- *yuuge* mistake!) or a special cable that allows micro-SD to USB storage
transfers.

Not only that, it was impossible to even wipe the device due to the
protected partitions! I then decided to try my luck with sideloading
the device.  Looking back I should have probably upgraded the recovery
software to the latest version, but
[a few issues](https://github.com/TeamWin/Team-Win-Recovery-Project/issues/247)
[didn't inspire confidence](https://github.com/TeamWin/Team-Win-Recovery-Project/issues/333).

I then decided to take a crack at sideloading the LineageOS update.

## Sideloading via adb

I'm not an expert in `adb` but apparently it is possible to
"side load" things into the phone using this method. I'm still
[not exactly sure what that technically means][sideload],
but I foolishly convinced myself it was worth a try after reading a
few light-on-info blog posts.

It was easy enough to sideload the update:

``` shell
ll2 :: ~ » adb sideload lineage-14.1-20170308-nightly-kiwi-signed.zip
Total xfer: 1.00x
```

Was that all I needed to a slick new phone operating system?

Of course not. But I wanted to see if it'd work at all, so I eagerly rebooted
the phone.

## com.android.phone has stopped working

The phone began to restart, and after some churning, it even prompted
me for my encryption password. I was actually thinking that this turkey
would run right out of the box.

Then it occurred to me that I had forgotten to sideload OpenGApps,
so I was quite pessimistic that actual apps would work - defeating
the point of having a "smart phone".

What I didn't expect, however, was the "black screen of death"
that awaited me about 30 seconds after unencrypting the device.
After that, a cascade of processes stopped working, including
very important-sounding ones, like `com.android.phone` and
the system UI (**!**). Looks like I was not successful on my first pass.

At this point I realized I was going to be Having Fun, so I got a
snack and put on a playlist, because this was not going to be a
quick project.

## Recovery mode

It took me awhile to figure out that booting into recovery mode on the
Honor 5X involves holding volume-UP and power (not down as the wiki
indicates).  After figuring that out, I figured I'd first try to
update the recovery mode software in case they enabled some feature to
decrypt the device during recovery.

They have [some instructions](https://twrp.me/devices/huaweihonor5x.html)
which I followed. I chose to try to install via TWRP itself: look
towards the middle section of that page `TWRP Install`.

I downloaded the
[latest version for my phone](https://dl.twrp.me/kiwi/twrp-3.1.0-0-kiwi.img)
and pushed it:

``` shell
max@ll2:~ $ adb push $HOME/Downloads/twrp-3.1.0-0-kiwi.img /sdcard/
[100%] /sdcard/twrp-3.1.0-0-kiwi.img
```

Per their instructions: go into recovery, click install, select the
image, click the pushed image, select recovery, and flash. Reboot
the phone.

After doing this, you'll be tempted to hold the recovery mode
keys, but don't do that - it just results in a boot loop.
Letting go will take you right back to the newly updated
TWRP recovery screen.

It looks like the software update didn't help: I'm still stuck
with an encrypted partition and no way to unlock it, and
a currently unusable phone. Next up: try to sideload both
the update and OpenGApps:

``` shell
max@ll2:~ $ adb sideload lineage-14.1-20170308-nightly-kiwi-signed.zip
Total xfer: 1.00x
max@ll2:~ $ adb sideload Downloads/open_gapps-arm64-7.1-pico-20170311.zip
Total xfer: 1.43x
```

Side note: the percentages for `adb` don't seem to match the progress
bar on TWRP's screen. I know progress bars are often just made up
graphics to reassure users that "stuff is working" but that's a little
sketchy.

Anyway, more importantly, things seemed to work, other than
the inability to access `/data` - presumably due to encryption.
That doesn't seem ideal.

Will it blend?

## Progress?

After rebooting, I now got to see my beautiful phone's home
screen.

For about a second.

Then, the warnings about apps closing started appearing, and
everything went black... again.  Hardly ideal. I decided to reboot
into recovery, which is about the only thing I've figured out how to
do reliably at this stage.

But wait: when I try to boot into recovery, I'm now prompted by a Huawei
eRecovery software splash screen. The hell? Did sideloading both apps somehow
annihilate TWRP? Or did the recovery mode keys change yet again?

I was somewhat tempted to give up, plow ahead with the Huawei software,
then just nuke everything once it had - hopefully - restored my phone to
*Working Condition*. At least it was an option. But I figured I'd try to
at least get TWRP to show up before folding.

Finally, I was able to get TWRP to show up by holding volume-UP and
power during the duration of the boot until TWRP shows up. Letting go
prematurely somehow invokes the Huawei recovery utility. Perhaps that
will be useful info.

Let's try once more with TWRP to get something going.

## Act with intent

I decided to use TWRP to, individually, wipe everything, but instead
of checking all the boxes at once, check them individually. In other words:

* Click wipe
* Click advanced wipe
* Select Dalvik / ART `=>` Swipe
* Select cache `=>` Swipe
* Repeat for everything related to the phone - except maybe MicroSD card if you
  value its contents

To my surprise, this actually worked. There were no issues, except an
odd error about an invalid .zip file when the wiping Dalvik.  I'm not
sure how or why it worked. But now I was very optimistic that I would
be able to normally install LineageOS and OpenGApps per the original
instructions, and hopefully, end up with a usable phone.

I had to repush them as I had wiped internal storage, using the same
`adb push` commands from earlier.

I then clicked Install, selected the lineage zip, then added the open
gapps zip to the queue. Somewhat eerily, the process failed when I checked
zip file verification. So I unchecked it, queued up both zips again,
waited, ... and hoped.

## Boom - outta here

And just like that, after some serious loading, I was greeted by a
setup screen much like the first time I put CyanogenMod on the device.

I finally have got a LineageOS enabled phone with OpenGApps pico.

## Wrapping up

Well, as one should expect with any major software transition,
switching things around wasn't the smoothest experience. But with a
little patience, some music, a cup of decaf coffee with a hint of soy
milk, and the Internet, I was able to upgrade to LineageOS - leaving
CyanogenMod behind for good.

I'll be forking and contributing the LineageOS wiki with things
that I found that may help others. Here's hoping the process
is not as cumbersome as the software update was.

[adb]: https://wiki.archlinux.org/index.php/Android#Android_Debug_Bridge_.28ADB.29
[sideload]: http://twrp.me/faq/ADBSideload.html
