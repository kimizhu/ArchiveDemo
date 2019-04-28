# JScript, Localization and Those Wacky Newfoundlanders

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/18/2004 12:25:00 PM

-----

I was talking about [localization in general](http://blogs.msdn.com/ericlippert/archive/2004/05/10/129481.aspx)the other day.  Today, some brief notes on localization in JScript Classic. 

A while back, a coworker asked me: 

> Does toLocaleString() do anything different than toString() when locale changes?  I am especially interested date formats when you run code in a place inside the United States that has weird Daylight Savings Time rules.

That's actually quite a complicated question.  I'm going to break that down into many sub-questions. 

What locale does JScript Classic use when it needs to change its output based on locale? 

**It depends on the host**.  JScript and VBScript have their own locale settings which may be **independent** from the host, user and machine settings.  A web browser host could, for example, set the script engine to be French if it were displaying a French web page.  

IE has changed their algorithm (both intentionally and accidentally) many times over the years.  Sometimes they used the locale of the user, sometimes they always defaulted to US-English.  I believe we are now in "always default to US-English" mode in IE.  It's confusing because the script engines have the ability to change the locale used for the **error messages** independent of the locale used to format dates, numbers, etc.  This allows for exotic scenarios like a VBScript host which displays French error messages while running scripts which use Japanese date formats. 

I am not aware of any hosts which actually use these features\!  We added them when IE4 shipped, but it turned out that IE didn't need the feature after all.  However, in the interests of completeness, I'm going to document JScript's behaviour as though there actually were a host that exposed this feature. 

How many methods are there in JScript intended for localization, and what are they? 

Nine.  On the String prototype there are three: toLocaleLowerCase, toLocaleUpperCase and localeCompare. The Date prototype has toLocaleDateString, toLocaleTimeString, and toLocaleString.  The Array, Object, and Number prototype objects each have an implementation of toLocaleString. 

Do any of these methods change their output when the script engine is in a different locale? 

Yes, eight of them do.  Of course, they depend on the **script engine locale**, which as I noted above may different from the user locale, depending on the whim of the script host. 

Object.prototype.toLocaleString simply calls toString.  No localization behaviour here unless of course you have overridden the prototype. 

Array.prototype.toLocaleString uses the operating system's localized list separator and recursively calls toLocaleString on each of its members to generate a localized, comma-or-whatever-your-locale-prefers separated list. 

Number.prototype.toLocaleString uses GetNumberFormat to get the number separator and decimal point.  If for any reason that fails, we fall back to OLE Automation's VariantChangeTypeEx, which is locale-sensitive. 

Date.prototype.toLocaleDateString, toLocaleTimeString, and toLocaleString are complicated by some bizarre weirdnesses in the Win32 NLS API.  To work around various problems, only dates between 1600 and 10000 AD are localized. Hebrew date formats for years after 2240 AD are also not supported.  Once we jump through those hurdles, the Win32 APIs GetDateFormat and GetTimeFormat are used to format the strings.  (I'm vaguely recalling that there was also a bug in there involving the Thai calendar but I don't remember the details.) 

String.prototype.toLocaleLowerCase, toLocaleUppercase, toLowerCase and toUpperCase are buggy -- they use the default user locale, not the script engine locale.  Looks like I was careless when I implemented the script engine locale feature.  I wouldn't expect those bugs to be fixed any time soon, since as far as I know, I'm the first person to find the bugs since I implemented them\! 

They just use LCMapCase to change the case of a string using the localized rules. 

String.prototype.localeCompare uses the script engine locale to call CompareString on two Unicode strings. 

What about Daylight Savings Time? 

The JScript runtime library always determines the local time zone offset by calling into various operating system APIs.  If the operating system gets the DST calculation right in whatever weird state you're in, JScript will also get it right.  If the operating system gets it wrong, complain to the Windows guys, not me. 

Speaking of that, exactly which of the United States have weird Daylight Savings Time rules? 

They are Alabama, Alaska, Arkansas, California, Colorado, Connecticut, Delaware, Florida, Georgia, Idaho, Illinois, western and eastern portions of Indiana, Iowa, Kansas, Kentucky, Louisiana, Maine, Maryland, Massachusetts, Michigan, Minnesota, Mississippi, Missouri, Montana, Nebraska, Nevada, New Hampshire, New Jersey, New Mexico, New York, North Carolina, North Dakota, Ohio, Oklahoma, Oregon, Pennsylvania, Rhode Island, South Carolina, South Dakota, Tennessee, Texas, Utah, Vermont, Virginia, Washington, West Virginia, Wisconsin and Wyoming.  

These bizarre states actually *legally mandate ***millions** of people to change their clocks twice a year in a confusing, pointless and error-prone light-saving gimmick. 

Arizona, Hawaii and central portions of Indiana have a single, sensible, simple DST rule: "we do not observe Daylight Savings Time." 

For what it's worth, My Home And Native Land doesn't fare much better.  All young Canadians grow up knowing the phrase "Coming up at **two-o'clock-two-thirty-in-Newfoundland**..."  

A half-hour timezone?  What the heck? 

Yep.  They're half an hour ahead of the rest of us in Newfoundland.  One year Newfoundland decided to try an experiment in "Double Daylight Savings Time" -- they sprang forward TWO hours.  Of course, they were already sprung forward half an hour as it was.  It didn't get dark until 11 PM. There was no end of problems. The failed experiment was described in the Newfoundland assembly a few years later like this: 

> We were up all night, all day; we could not get to sleep. You could not get a coffee in the morning at Tim Horton's because there was no coffee; there was no morning. It was just going around - the sun never set.  Newfoundland - the place where the sun never set. I think the Minister of Environment at that time was the honourable John Butt.  I believe he led the way in that great endeavour. He had Newfoundlanders going around - they did not know whether they were asleep, whether they should go to bed, whether they should get up. I think it was at 1:00 a.m. that we used to get a little bit of dusk, and before you knew it, it was daylight again. John even had the sun running the way he wanted it. He had the sun slowed down. That is how the Tories operated. He said he did that after consultation with the people. The only thing he consulted with must have been a crystal ball or something. He must have looked into a crystal ball, because before half the summer was out, the Premier of the day was trying to have the time switched back again.

Somehow I managed to segue from localization rules to Newfie partisan politics; this would probably be a good time to stop.

