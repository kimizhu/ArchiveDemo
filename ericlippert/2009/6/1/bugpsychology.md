# Bug Psychology

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/1/2009 9:48:00 AM

-----

Fixing bugs is hard.

For the purposes of this posting, I’m talking about those really “crisp” bugs -- those flaws which are entirely due to a failure on the developer’s part to correctly implement some mechanistic calculation or ensure some postcondition is met. I’m not talking about oops, *we just found out that the product name sounds like a rude word in Urdu*, or *the specification wasn’t quite right so we changed it* or the code wasn’t *adequately robust in the face of a buggy caller*. I mean those bugs where you were asked to compute some value and you just plain get the result *wrong* for some valid inputs.

Let me give you an example.

The first bug I ever fixed at Microsoft as a full-time employee was one of those. To understand the context of the bug, [start by reading this post from the early days of FAIC, and then come back.](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53013.aspx)

Welcome back, I hope you enjoyed that little trip down memory lane as much as I did.

Now that you understand how a VT\_DATE is stored, that explains this bizarre behaviour in VBScript:

print DateDiff("h", \#12/31/1899 18:00\#, \#12/30/1899 6:00\#) / 24  
print DateDiff("h", \#12/31/1899 18:00\#, \#12/29/1899 6:00\#) / 24

This prints –1.5 and –2.5, as you’d expect. There’s a day and a half between 6 AM December 30th and 6 PM December 31st, and two and a half days between the other two dates. This is perfectly understandable. But if you just subtract the dates:

print \#12/31/1899 18:00\# - \#12/30/1899 6:00\#  
print \#12/31/1899 18:00\# - \#12/29/1899 6:00\#

You get 1.5 and 3, not 1.5 and 2.5. Because of the bizarre date format that VT\_DATE chooses, when you convert dates to numbers, you cannot safely subtract them if they straddle the magic zero date. That’s why you need the helpful “DateDiff”, “DateAdd” and so on, methods.

The bug I was assigned was that testing had discovered a particular pair of dates which DateDiff was not subtracting correctly. I took a look at the source code for one of the helper methods that DateDiff used to do one of the computations it needed along the way. To my fresh-out-of-college eyes it looked something like this:

if (frob(x) \> 0 && blarg(y)) return x – y;  
else if (frob(x) \< blarg(y) && blah\_blah(x) \> 0 || blah\_de\_blah\_blah\_blah(x,y)) return frob(x) – x + y + 1;  
else if…

There were seven such cases.

My urge was to dive right in and add an eighth special case that fixed the bug. But my ability to get it right in the face of all this complexity concerned me. It seemed like this was an awfully complicated function already for what it was trying to do.

I researched the history of the code a bit and discovered that in fact variations on this bug had been entered… seven times. Each special case in the code corresponded to a particular bug that had been “fixed”, a term I use guardedly in this case. A great many of those “fixes” had actually introduced new bugs, regressing existing correct behaviour, which then in turn were “fixed” by adding special cases on top of the broken special cases that had been added to “fix” previous bugs.

I decided that this coding horror would end here. I deleted all the code (all seven lines of it\! I was bold\!) and started over.

Deep breath.

Spec the code requirements first. Then design the code to meet the spec. Then write the code to the design.

Spec:

  - Input: two doubles representing dates in VT\_DATE format.
  - VT\_DATE format: signed integer portion of double is number of days since 12/30/1899, unsigned fractional part is portion of day gone by. For example: –1.75 = 12/29/1899, 6 PM.
  - Output: double containing number of days, possibly fractional, between two dates.  Differences due to daylight savings time, and so on, to be ignored.

Design strategy:

  - Problem: Some doubles cannot simply be subtracted because negative dates are not absolute offsets from epoch time
  - Therefore, convert all dates to a more sensible date format which can be simply subtracted.

Code:

double DateDiffHelper(double vtdate1, double vtdate2)  
{  
  return SensibleDate(vtdate2) – SensibleDate(vtdate1);  
}  
double SensibleDate(double vtdate)  
{  
  // negative dates like –2.75 mean “go back two days, then forward .75 days”:  
  // Transform that into –1.25, meaning “go back 1.25 days”.  
  return DatePart(vtdate) + TimePart(vtdate);  
}

I already had helper methods DatePart and TimePart, so I was done. The new code was shorter, far more readable, generated smaller, faster machine code and most important, was **clearly correct**. No special cases; no bugs.

It’s not that my coworkers were dummies. Far from it. These were smart people. But computer geek psychology is such that it is very easy to narrow-focus on the immediately wrong thing, and try to tweak it until it does the right thing.

When faced with these sorts of “crisp” bugs, I try to restrain myself from diving right in. Rather, I try to psychoanalyze the person – who is, of course, usually my past self – who caused the bug. I ask myself “how was the person who wrote the buggy code fooled into thinking it was correct?” Did they not have a clear specification of what the method was supposed to do? Was it misleading? Did they have a clear plan for how to proceed? If so, where did it go wrong?

If there never was either a spec or a plan, then for all you know the whole thing might only be working by sheer accident. There could be any number of design flaws in the thing that just haven’t come to light yet. Editing such a beast means adding unknown to unknown. which seldom leads to good results. Sometimes coming up with a new spec, a new plan and scrapping an existing bug farm is the best way to proceed.

For many years after that, I would ask how to implement DateDiffHelper as my technical question for fresh-out-of-college candidates that I was interviewing for the scripting dev team. I reasoned that if that was the sort of problem I was given on my first day in the office, then maybe that would be a reasonable question to ask a candidate.

When you ask the same question over and over again, you really get to see the massive difference in aptitude between candidates. I had some candidates who just picked up a marker, wrote a solution straight out on the board, wrote down the test cases they’d use to verify it, mentally ran a few of the tests in their head, and then we’d have another half hour to chat about the weather. And I had some candidates who tried earnestly to write the version using special cases, despite my specifically telling them “you might consider transforming this bad format into something more pleasant to work with”, after they got stuck on the third special case. I’d point out a bug and immediately they’d write down code for another special case, rather than stopping to think about the fact that they’d just written buggy code three times already and told me it was correct three times.

