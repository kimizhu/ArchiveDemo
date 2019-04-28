# Eric's Blog for January 279th, 2003

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/6/2003 2:08:00 PM

-----

 

I'm having my kitchen redone.  

Yes, I remember that [in my very first blog entry](/ericlippert/archive/2003/09/12/52975.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/12/52975.aspx") I said that I would be talking about only technical matters, not trivia about my life.  Bear with me, people. 

When I say "redone" I mean that I took a crowbar, a sledgehammer and a couple of helpful housemates, pulled down all the walls, pulled up the floor, trucked the asbestos-laden vinyl flooring to the dump and then invited over a bunch of big burly guys to build a new kitchen from scratch. 

Unfortunately, we found a number of "structural issues" once the walls were apart which necessitated some pretty major infrastructure improvements.  I was hoping to have all the work in the kitchen done in early October, but these various non-scheduled improvements ate up a lot of time.  I got an email from the general contractor the other day saying that the revised schedule now has us finishing up in October -- October 45th to be precise.  Ha ha ha.  Very funny guys.  

(Update: We finished on January 16th, taking exactly twice as long as initially forecast, and confirming the old saying that a poorly planned project takes three times as long as you think, a well planned project only twice as long.) 

The good news is that this reminded me that when I dissed the VBScript date format a few weeks ago, I never got around to dissing the similarly goofy JScript date code.  **October 45th?  No problem\! **

****

var d = new Date(2003, 9, 45);  
print(d); 

prints out 

Fri Nov 14 00 :00:00 PST 2003 

**What the heck?  "2003, 9, 45" is the 14th of November?  **

Yep.  First of all, for reasons which were never adequately explained to me by my cohorts at Netscape, the date constructor numbers dates starting from zero.  So "9", not "10", is October.  If you want to know why that is, ask Brendan next time you see him, 'cause I sure don't know. 

But what's more interesting is that any attempt to schedule projects like my contractor made result in real dates.  November 14th is, in some sense, the best candidate for October 45th, as it is 44 days after October 1st.   You can get weirder than that, of course.  

var d = new Date(2003, -1, 0); 

prints out 

Wed Nov 30 00:00:00 PST 2002 

Which is only logical -- if January was month zero of 2003 then December 2002 was the "negative first" month of 2003.  And if December 1st was the 1st day of December, then November 30th must have been the 0th day (and, incidentally, my birthday.)  Similarly, November 29th was the -1st day, and so on. 

The underlying implementation details of JScript dates are quite a bit more sensible than VBScript's implementation.  JScript also stores dates as double-precision 64 bit floats, but the JScript implementation stores all dates as the number of milliseconds since midnight, 1 January 1970 UTC.  (Universal Time Coordinated is the proper name for what most people call "Greenwich Mean Time" -- the time at the prime meridian, with no adjustment for British Summer Time.) 

For all practical purposes, we treat the 64 bit float as a 53 bit integer plus a sign bit.  This means that JScript has 300000 years of millisecond accuracy on either side of the epoch, as opposed to VBScript where the precision varies over the range.  Also, in this system there are no two dates with the same numeric representation, which makes comparing, subtracting, adding and rounding times straightforward.  

Also, JScript, unlike VBScript, adjusts for daylight savings time and time zone differences according to the local information reported by the operating system. 

There are some weirdnesses in JScript though.  The most bizarre is the getYear method, which returns a one-or-two-digit number for all dates from 1900 to 1999, and a four-digit number for dates before 1900 or after 2000.  Again, why Javascript has a function that obviously causes Y2K-style errors implemented a few years before Y2K is unknown to me -- ask your favourite Netscape language designer next time you meet one at a party.  Microsoft didn't make the problem any better when we screwed up our implementation of getYear -- the original IE implementation returns the year minus 1900, so 2000 was year 100.  Fortunately we corrected that incompatibility in the second version.  **You should never use ****getYear -- use getFullYear, which always returns all the necessary digits. **

More generally, you might wonder if there are design principles underlying the decision to make "October 45th" a legal date.  Indeed there are (though to be perfectly frank, when it comes to the date handling, I suspect that any "justification" might be somewhat post hoc.) 

Here are some thoughts that go through the minds of language designers: 

  - The sooner you produce an error for bogus behaviour, the sooner the developer will catch the bug.  Therefore, **at the first sign of anything out of the ordinary, crash and die immediately. **
  - But wait a minute -- the more errors we produce, the more error handling code the developer will have to write, because some of those bogosities will be produced by end user data. 
  - Worse, if the developer misses an error handling case then the error will be reported to the end user. 
  - The end user of a web page script is someone who has absolutely no ability to understand the error.  If they do understand the error, they have no ability to go to the web server and fix it.  Therefore, **never produce error messages unless you absolutely have to.** 

The ability to start with a premise and deduce its opposite is a neat trick, you must admit\!  But seriously, the existence of conflicting goals explains why there is more than one programming language. Hard-core application development languages like C\# are designed to trap as many bugs as possible at compile time and run time so that you can find them, write the handling code, and ship.  C\# demands that you type your variables, that you call reflection explicitly, that you mark unchecked arithmetic, and so on.  

But C\# is a language designed for building hundred-thousand-line application frameworks and object models, not hundred line web page scripts.  C\# is designed for professional developers, not hobbyists and departmental web developers who just want to get "glue code" written as rapidly as possible. 

That's why JScript's attitude towards error cases is not "die at the first sign of trouble" but rather "muddle along as best you can".  Assign to an undeclared variable?  *Declare it dynamically.*  [Forget a semicolon?  *Insert it.*](/ericlippert/archive/2004/02/02/66334.aspx "http://blogs.msdn.com/ericlippert/archive/2004/02/02/66334.aspx")  Try to create a date for the 31st of September?  *Move it* to October 1st.  Divide by zero?  *Return a NaN *.  Reference an element beyond the end of an array?  *Grow the array*.  All these things that would be compile time or runtime errors in other languages are just handled for you in JScript, for better or for worse. 

And this is why when people come to me and say "*my team of twenty developers has written a hundred-thousand-line database program in JScript and now I can't make head nor tail of it, help\!"* [I tell them to use the right tool for the job](/ericlippert/archive/2003/11/18/53388.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/18/53388.aspx")\!  C\# was designed for those tasks.  Heck, JScript .NET was designed for those tasks\!  But a **weakly-typed, interpreted, fail-silently, late-bound, non-generational-GC **language like JScript was designed to add scripting to simple web pages.  If I give you a really great hammer then you can build a decent house but not a decent skyscraper.

