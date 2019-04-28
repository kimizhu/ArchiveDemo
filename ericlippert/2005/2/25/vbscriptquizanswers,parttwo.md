# VBScript Quiz Answers, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/25/2005 2:29:00 PM

-----

When you're getting close to shipping a beta for a big product, the dev team is either insanely busy tracking down last-minute issues, or deathly bored. This week was one of the insanely busy weeks\! Lots of late nights. It looks like next week should be a little calmer though. Anyway, on with the quiz answers. 2) Which of the following are Integer, not Long? Why? (a) w = TypeName(32768)  
(b) x = TypeName(32767 + 1)  
(c) y = TypeName(-32767-1)  
(d) z = TypeName(-32768) (c) is Integer. The rest are Long. I actually gave the answer to this one already, [here](http://weblogs.asp.net/ericlippert/archive/2004/07/29/200902.aspx). The range of a short integer is -32768 to 32767, so it should be clear why (a) and (b) are Long. (c) is a short integer minus a short integer, the result of which fits into a short integer, so it's a short integer.  But (d) is the unary minus operator applied to a long integer, so it's aLong\! Incredibly, (a), (c) and (d) agree with VB6. but (b) throws an overflow error in VB6\! VBScript automatically overflows into larger types but VB6 does not. So this is yet another violation of the subset property. One reader asked if this meant that there were no negative literals in VBScript. Indeed, that is a reasonable interpretation. But in a later question we'll show that it's not *quite* that simple. (Incidentally, VBScript and VB.NET both overflow into larger types, but they use different overflow resolution algorithms. That's a subject for another blog though.)

