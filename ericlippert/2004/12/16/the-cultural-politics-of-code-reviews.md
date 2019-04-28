<div id="page">

# The Cultural Politics of Code Reviews

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/16/2004 11:58:00 AM

-----

<div id="content">

For the last few weeks we've been doing a line-by-line code review of everything we've written in the last two years. We're doing it in pairs, interestingly enough. There are many judgment calls to make when you do that kind of thing -- on the one hand, you don't want to touch working code and risk checking in a breaking change accidentally. On the other hand, some of that code is really quite crufty and could benefit from refactoring and general cleanup. I think we'll err on the side of caution\! But that's not what I wanted to mention today. Yesterday my coworker Geoff and I were going through some code I wrote: Eric: "… so this PROPVARIANT structure here is a discriminated union, and --"  
Geoff: "A *discriminated union?* Like, gay marriage?" Yes, *exactly* like that. I've often thought recently how unfortunate it is that structures leading a VARIANT lifestyle are stuck in a discriminated union<sup>\*</sup> even in the enlightened 21<sup>st</sup> century. They're still being repressed by traditional nondiscriminated unions like DECIMAL and CURRENCY and LARGE\_INTEGER. That's why you see so many VARIANTs talking about moving to Canada lately. And with that silly joke, I'm off to my ancestral home (in VARIANT-friendly Canada) for my winter vacation. I will have network access, and I might blog a little, or, maybe not. If not, let me take this opportunity now to say thanks to all the readers and commenters out there for your support and questions and interest over the last year. I'll catch you in the new year\!     (\*) For those of you who are not C programmers, I should explain that joke. A *union* is a structure that allows you to give the same chunk of memory two or more names. That then allows you to create memory-efficient structures that have the semantics of "this structure contains either a string or a date, but not both", for instance. A union in which there is a special "flag" that tells you which name is valid right now is called a "discriminated union". In a VARIANT, for example, you can access var.bstrVal only if var.vt == VT\_BSTR. If var.vt == VT\_DATE, accessing var.bstrVal will almost certainly lead to bad consequences\! The vt stands for "variant type" -- it *discriminates* which names in the union are currently valid. In a *nondiscriminated* union, either it's the case that all names are always valid, or there's some other way to figure out what the currently valid set of names is.

</div>

</div>

