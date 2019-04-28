# Eric's Complete Guide To VT\_DATE

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 2:50:00 PM

-----

I find software horology fascinating.

The other day, [Raymond said](http://blogs.msdn.com/oldnewthing/archive/2003/09/05/54806.aspx) "The OLE automation date format is a floating point value, counting days since midnight 30 December 1899. Hours and minutes are represented as fractional days."

That's correct, but actually it is a little bit weirder than that. I suspect that I may be the world's leading authority on bugs having to do with the OLEAUT date format, a dubious distinction at best.

Here are some interesting (well, interesting to me) facts about OLEAUT dates.

First of all, let's start with the obvious problem -- Midnight 30 December 1899 in what time zone? We never say. OLEAUT dates are always "local" which makes it very difficult to write code that uses OLEAUT dates -- any VB or VBScript program, for example -- which must deal with two things happening at the same time in different time zones.

Next, what about daylight savings time? How does one represent those days which due to springing forward or falling back have 23 hours or 25 hours? Again, those who use the OLEAUT date format need to pretend that these days do not exist.

Now let's get really weird. An OLEAUT date is, as Raymond noted, a double where the signed integer part is the number of days since 30 December 1899 and the fraction part is the amount of that day gone by. So what is 1.75? That's 6PM , 31 Dec 1899. What about -1.75? That's 6 PM 29 Dec 1899.

What about 0.75 and -0.75? Uh, those are zero and "minus zero" days from 30 December 1899, 6 PM . Those are the same time. This means that any program which must calculate the difference between two OLEAUT dates must say that -0.75 - 0.75 = 0.

The reason I know all this is because my first ever checkin as a full timer was a rewrite of the VBScript implementations of DateAdd and DateDiff, both of which used to be a mass of spaghetti code to handle all the special cases entailed by the discontinuities between -1.0 and 1.0. Incidentally, I now ask interview candidates to write me those algorithms during interviews\! (Attention potential interview candidates reading this: writing a mass of spaghetti code on my whiteboard is a bad idea. Solving this problem by cases is a bad idea. There are better ways to solve this problem\!)

Here's another bogus one: how about -1.99999999 and -2.0? Those are 0.00000001 apart in numbers but almost 48 hours apart in time\! But OLEAUT dates are rounded to the nearest half second by the operating system, and you guessed it: any dates less than a quarter second before midnight before 30 Dec 1899 are sometimes "rounded" two days wrong. I wrote the code that converts OLEAUT dates to JScript dates, and it at least does correctly handle this case, though I had to jump through some hoops to do it.

Some more oddities: The range is enormous, and the precision varies greatly over the range\! You can represent dates long before the creation of the Universe, though you lose precision as you go. The only valid dates for the date format are between 100 AD and 10000 AD, and since we have half-second granularity, we are wasting a whole lot of bits here.

And finally, why 30 December 1899? Why not, say, 31 December 1899, or 1 January 1900 as the zero day? Actually, it turns out that this is to work around a bug in Lotus 1-2-3\! The details are lost in the mists of time, but apparently Lotus 1-2-3 used this date format but their devs forgot that 1900 was not a leap year. Microsoft fixed this bug by moving day one back one day.

Getting date code right is hard this is one of those areas where messy human requirements are hard to translate into crisp machine logic. There are huge localization problems for instance, in Japan it is legal to specify years as the fifth year of the reign of Emperor Hirohito . Thailand s calendar doesn t count from 1 AD. In Israel , the start and end days for daylight savings time are not standardized but rather are declared anew by the government after fierce debate every year.

These situations are fraught with peril for the unwary developer. I once drove our Microsoft support team in Israel to distraction by accidentally changing both Arabic and Hebrew locales to display dates right-to-left apparently Arabs really do read dates right-to-left but Israelis use left-to-right dates even when they are embedded in right-to-left Hebrew text.

You wouldn t think that something as simple as what day is it? could lead to so many problems, but the world is seldom as cut-and-dried as we software developers would like.

UPDATE: Some more history on this bizarre date format can be found in [this article by Joel Spolsky.](http://www.joelonsoftware.com/items/2006/06/16.html)

