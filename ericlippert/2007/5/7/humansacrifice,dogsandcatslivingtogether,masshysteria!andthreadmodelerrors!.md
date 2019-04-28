# Human sacrifice, dogs and cats living together, mass hysteria\! and thread model errors\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/7/2007 7:22:00 PM

-----

Reader Shaka comments on [my post about error messages](http://blogs.msdn.com/ericlippert/archive/2006/07/07/659259.aspx) that "catastrophic failure" really does take the cake as being a terrible error message.

I fondly remember the first time I saw "catastrophic failure" as an error message. I was an intern, running the build lab for Visual Basic for Applications, and I was trying to get the PowerPC cross-compiler working so that we could build PPC binaries on x86 hardware. (Note that this was in the early 1990's. The VS build lab is now an entire team of people who have dedicated circuits just for the cooling equipment for all their computers. We've come a long way from an intern, a closet, and six machines named after opponents of Godzilla (\*).)

Anyway, the compiler had a bug which caused it to run out of stack while compiling a particular module and the error was "catastrophic failure". This amused me to no end at the time. I laughed out loud, literally. I mean, *fatal*, sure. That process is going *down*. But "catastrophic"? I expect catastrophes to at least result in my hard disk becoming unreadable. I want bridges collapsing and floods and [mass hysteria](https://www.youtube.com/watch?v=O3ZOKDmorj0)\!

I have therefore always been rather embarrassed that the error message you get when you call the script engine on the [wrong thread](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx) is "catastrophic failure". That, unfortunately, is the somewhat breathless and overstated string associated with E\_UNEXPECTED. Basically we want to say "you called us in a completely unexpected way which thoroughly violates the engine contract, please don't do that". But "catastrophic failure" is what you get. I wish now that we'd declared a new [HRESULT](http://blogs.msdn.com/ericlippert/archive/2003/10/22/53267.aspx) just for "you've violated the engine contract". But it's far, far too late now.

(\*) In the interests of total accuracy: the OLE Automation build machines were named after Godzilla's opponents: Mothra, Monster Zero, etc. The VBA build machines were named after species of marine mammals: sei, humpback, etc.

