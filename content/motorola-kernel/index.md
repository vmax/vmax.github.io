---
title: That day when I felt like a kernel hacker
date: 2021-03-14
description: and a note on sustainable technology
---

Circa 2018 I used to rock a cheap-but-reliable smartphone: [Moto G5s](https://www.gsmarena.com/motorola_moto_g5s-8698.php).
On a fateful day, it slipped through my pocket and ended up on the ground. Of course, the screen cracked and I
had no other choice but to take it to the repair shop. It quoted me a _third_ of the price
I originally paid for the whole thing and I accepted.

Everything went fine until an update to Android 8.1 Oreo arrived. My touchscreen, perfectly
operational before the update, stopped functioning!

Being somewhat comfortable playing around with Android ROMs, I unlocked the bootloader and rolled the operating system back
to Android Nougat. From now on, I was puzzled: I wanted to have the latest-and-greatest operating system
but it would also imply the most essential part of the smartphone would not work.

Android is based on Linux, an open-source software. More importantly, it's _copyleft_ software which means
anybody who modifies it must give back modifications to the mainline source tree. Motorola was no exception and
so I managed to find the sources [here](https://github.com/premaca/kernel_motorola_msm8953_sanders).

Deductive reasoning led me to realize that the very code that made my phone respond (or not respond) to touches
must be stored under `drivers/input/touchscreen` source directory. What I proceeded with doing is grabbing the
full directory from  `nougat-7.1.1-release-sanders` branch, switching to `8.1-moto` branch and copying the old version
into the source tree.

Imagine my surprise when it **worked**. My phones started responding to touches again _and_ it was running latest-and-greatest
Android Oreo. I was happier than a little child in a candy store! I also managed to snug in my custom branding :)

![Motorola kernel](https://user-images.githubusercontent.com/499267/111919188-b8ccf000-8a99-11eb-93b4-1751383ea00e.png)

Now, the sustainability bit that I teased. I once had to do something similar (downgrading) to my friend's iPhone 6S.
After upgrading, the display worked but the touchscreen stopped responding (apparently repair shops get the screen from very untrustworthy vendors!). However, there's no such thing as unlocking the bootloader at your own will in the Apple world.
What you can do is downgrade while the firmware you're trying to downgrade to is still _signed_ by Apple (you can check it [here](https://ipsw.me/)).
Luckily for my friend, he upgraded _before_ they stopped signing the old version and we were able to roll it back
to the version that still worked. Given the closed nature of Apple, that's as far as we can get. We cannot hack it around to use a mishmash of old touchscreen driver and new operating system. Once upgraded, it shall remain a glass brick forever and ever.
This Android phone, on the other hand, still runs a modified kernel to this day with no reliability issues.
