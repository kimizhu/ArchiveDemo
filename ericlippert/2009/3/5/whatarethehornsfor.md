# What are the horns for?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/5/2009 12:40:53 PM

-----

(Technology of a different sort today, just for a change of pace.)

[![FalkirkWheel](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Whatarethehornsfor_8805/FalkirkWheel_3.jpg)](http://en.wikipedia.org/wiki/Falkirk_Wheel) The first time I saw a picture of the Falkirk Wheel -- the world's only rotating boat lift, in Scotland -- I thought that it must be a really nice computer-generated landscape. It looks like something you'd see in Halo.

But it's real; it lifts and lowers boats 24 metres between two canals. I was first sent photos of it by some engineer friends of mine, and we got into an email discussion of some of the interesting points in the design and implementation of such a device.

First off, does the weight of the boats in the rotating caissons matter? What if one caisson has a lot of heavy boats and the other has only light boats -- won't the imbalance put extra stress on the structure and the gears?

No\! Boats displace their own weight in water -- at least, boats that still float do. Therefore when a boat of any tonnage enters a caisson, a mass of water equal to the mass of the boat leaves. As long as both caissons have the same volume and are filled to the same height, they always have the same mass no matter what boats are in them.

This means that it is straightforward to keep the imbalance between the two sides pretty small, just by using pumps to ensure that each caisson is filled to the same height. Because the system is so well balanced, and takes a good seven minutes to rotate, [the power requirements are surprisingly low](http://www.gentles.milestonenet.co.uk/fcucanalweb/BEK/statistics.html). In the six-to-twelve kilowatts range, which is only about one hundred bright light bulbs. (People are often surprised by how little power it takes to slowly rotate something balanced. For example, apparently the motor that powers the rotating restaurant in the Space Needle is a one-horsepower motor with a *huge* gear ratio.)

 [![FalkirkGear](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Whatarethehornsfor_8805/FalkirkGear_3.jpg)](http://www.gentles.milestonenet.co.uk/fcucanalweb/technicaltour/technical.html)We then got into a lively discussion of what the horns on the sides of the Wheel rotor are for. One of my engineer friends thought that one of the horns might be deliberately heavier than the other, the resulting torque providing some mitigation of the "play" or "[backlash](http://en.wikipedia.org/wiki/Backlash_\(engineering\))" inherent in all gearing systems. But as you can see if you look carefully in the photo, the designers of the Wheel came up with a different solution to mitigate backlash in the planetary gears. **The "teeth" of the lower gear are actually rollers.** The gears fit together the same way that a bicycle sprocket fits the rollers in the chain. Neat\!

This still doesn't explain what the horns are for. After lots more speculation I decided to cut to the chase and emailed the public relations people at the Wheel. They cheerfully informed me that the horns were entirely decorative, served no engineering purpose, and were designed to be reminiscent of Celtic-style axes.

I don't believe them. I know what the horns are for. I mean, think about it. When the descending side of the Wheel strikes the water, **the sharp horn breaks the surface tension** and allows the rest of the rotor to *smoothly* enter the pool below. That's the only sensible explanation.

(Wikimedia Commons photo by Sean Mack, gear photo by James Gentles.)

