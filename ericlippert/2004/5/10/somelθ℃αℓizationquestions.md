# Some LΘ℃αℓization Questions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/10/2004 6:47:00 PM

-----

A reader asked me a few questions about localization the other day.  That's not a subject that I have a lot of experience on, but I can speak to it a bit.  (I know that I've seen a blog from a Microsoft localization PM *somewhere* in the last six months, but cannot for the life of me remember who, and my google-fu has failed me.  Anyone know about whom I'm thinking?)   If your employer does not have a "current need" but maybe a "future need" for internationalization, how far do you go while coding? 

 Hard question.  Accessibility and localization have a lot in common; both are about making software usable by people who may have quite different UI needs than the "typical" user.  My friend and world-famous accessibility guru [Matt](http://www.bestkungfu.com/ "http://www.bestkungfu.com/") has [often](http://www.bestkungfu.com/archive/?keyword=accessibility "http://www.bestkungfu.com/archive/?keyword=accessibility") made the point that accessibility is not a "feature" that can be added on post hoc, but rather something that has to be baked in.  Well, the same goes for localization. 

 It's a hard question because it really depends a lot on what you're doing.  I mostly write dev tool back end systems -- compilers and code generators and whatnot. I leave the UI to other people.  Since my code mostly turns one kind of string (source code) into another kind of string (machine code), I do not require much localization support.  Some, yes.  I make sure that all the error strings are in resource files and all the string manipulation plays well with Unicode, and I'm pretty much done.  The people who write the tooltips and help system and all the other UI-heavy stuff have a lot more internationalization worries. 

 Do you hard code strings until the class unit tests OK, then go back and add them to a resource file? 

 I wouldn't.  Sounds like more work than doing it right in the first place.  

 Something we did when we started our current project -- I mean, like, day one of coding -- was wrote up a dead simple class that was just a "string server".  We give it a magic constant, it gives us back a string.  That meant that we were then free to figure out later how the strings would actually be stored.  They started off as hard-coded in the string server class for the first couple days, and then we figured out how to get it all working in resource files.  Make it flexible enough and you'll be able to change the underlying storage without disturbing all the string consumers. 

 Do you always/sometimes/never calculate UI layout fields using MFC's GetTextExtent() or similar? 

 I haven't written an MFC app for a long, long time.  But how else would you do it?  What, just guess at how big it's going to be, and hope that you never change the text, font, size, layout, etc?  It seems fundamentally brittle, and hence more work, to make assumptions.  ([Raymond](http://weblogs.asp.net/oldnewthing/ "http://weblogs.asp.net/oldnewthing/") talked about a similar issue with [determining rectangle extents](http://weblogs.asp.net/oldnewthing/archive/2004/02/17/74811.aspx "http://weblogs.asp.net/oldnewthing/archive/2004/02/17/74811.aspx") a while back.) 

 Do you think about right-to-left reading languages up front? 

 Oh yeah -- any time there is the potential for bidirectional text, it pays to think about it early.  Something we did in the script engine syntax colouring code for instance was add a bit that basically meant "this chunk of text is probably a human-readable string, and therefore the dev environment needs to figure out whether it is an RTL or LTR string."  (I just got a bug on that code a couple weeks ago, actually, which is why it immediately comes to mind.) 

 Is the UI abstracted to a resource DLL along with the string table? 

 We did in WSH.  I'm not sure that I could really speak to the pros and cons.  Like I said, user interfaces are not my strong suit. 

 Does the average programmer at Microsoft just do their thing and pass the code down to an internationalization expert who then adapts the code? 

 No, all the devs are responsible for writing the localization code.  The actual translation of the resources into foreign languages is done by experts of course.  Most teams have at least one "loc PM" and "loc tester" who are a good people to know when you have technical problems, need to track down bugs that only repro on the Korean build, etc.  

 We often do what we call "pseudolocalization" builds as easy sanity checks.  That is, we run the resources and whatnot through a pseudolocalizer, which replaces all the English text with stuff that is still readable, but is not normal English.  For instance, it might replace an instance of u0041 (LATIN CAPITAL LETTER A) with u24B6 (CIRCLED LATIN CAPITAL LETTER A).  It replaces every letter with a similar-looking letter in some random Unicode range, makes strings longer (to see if things fall off the ends of dialog boxes) by padding stuff out, bumps up font sizes in dialogs, etc. 

 Then we do a test run and see what breaks, what looks godawful, etc. But since it is only pseudo-localized, we can still read the dialog boxes and error messages and whatnot without running down the hall to find a colleague who speaks Bulgarian.  (Not that you'd have to go far; my team just hired our third Bulgarian speaker last week.)  Using pseudolocalization is way, way faster and easier than actually sending the bits to Ireland and Japan for localization and then testing the real-localized bits. 

 And something you didn't mention, but I'll call out right now: it is much more expensive to localize bitmaps than text, and very hard to make them accessible.  If you care about localization, don't go putting words onto bitmaps.  For that matter, don't go putting any picture on there that only makes sense to North Americans (like, say, most traffic signs.) 

 This is a topic that no green programmer is aware of (that I ever found), and many experienced developers don't even consider. 

 One of the reasons that Microsoft has been successful is very straightforward: we look for things that make people NOT use our products, and try to eliminate them.  As I said in an [earlier blog entry](http://weblogs.asp.net/ericlippert/archive/2003/10/28/53298.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/28/53298.aspx"), we care about legally blind Catalan-speaking customers.  If we didn't, they wouldn't be customers. 

 You are correct, and I would add security and accessibility to the list of things that some developers never think about, which is too bad. None of those things are easy to add post hoc, and all of them are barriers to entry.  We're trying to get security, accessibility and localizability baked into the framework itself -- nothing will make these things *easy*, but we can at least make it a little less mind-bogglingly difficult. 

 But, like I said, I am far, far from an expert on localizability.  If you have questions for a real expert, go ask [Doctor International](http://www.microsoft.com/globaldev/DrIntl/default.mspx "http://www.microsoft.com/globaldev/DrIntl/default.mspx"), or read the good doctor's [book](http://www.microsoft.com/MSPress/books/5717.asp "http://www.microsoft.com/MSPress/books/5717.asp").

