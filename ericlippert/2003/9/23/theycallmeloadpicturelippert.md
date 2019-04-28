# They call me "LoadPicture Lippert"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/23/2003 8:30:00 PM

-----

As promised, my *least* customer impactful bug ever.   

 

 

VBScript version 1.0 was written by one of the VB compiler devs, one of those uber-productive guys who implements compilers on the weekend for fun.  It was implemented and tested extremely rapidly, and as a result, a few of the less important methods in the VB runtime were not ported to the VBScript runtime.  My first task as a full timer at Microsoft was to add the missing functions to VBScript 2.0.

 

 

On August 5th, 1996, I implemented LoadPicture, which, as you might imagine, extracts a picture from a storage.  Here's a scrap of the code I wrote:

 

 

IDispatch \* pdisp;

// \[... open storage and stream ...\]

hresult = OleLoadPicture( pstream, 0, TRUE, IID\_IPicture, (void \*\*)\&pdisp );

 

 

Did I mention that I'd been writing COM code for all of two weeks at the time?

 

 

There's a boneheaded bug there.  I'm asking for an IPicture and assigning the result to an IDispatch.  This is going to crash and burn the moment anyone tries to call any method on that picture object because the vtable is going to be completely horked.  I've violated one of the Fundamental Rules of COM. The fact that this bad code shipped to customers indicates that:

 

 

\* Most seriously, I did not adequately test it before I checked it in

 

\* my mentors did not adequately review the code

\* the test team did not adequately test it before it shipped to customers

 

 

Those are all bad, and I am happy to say that today we are much, much more hard-core about peer-reviewing, self-testing, buddy-testing, and tester-testing code before it goes to customers than we were seven years ago.

 

 

But there is a silver lining of a sort -- obviously no one at Microsoft bothered to run this code after I wrote it.  But **neither did any customers**.  We didn't get a bug report on this thing until **February of 1998**.  This thing was in the wild for almost a year and a half before someone noticed that it was completely, utterly broken\!  No one really cared.

 

 

The next day the name plate on my office door said "LoadPicture Lippert".  Ha ha ha, very funny guys.

