# What could numeric rounding possibly have to do with MS-DOS?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/26/2003 7:10:00 PM

-----

A reader points out that FormatNumber uses yet a different rounding algorithm.  FormatNumber rounds numbers ending in .5's away from zero, not towards evens. He also asks whether FormatNumber, FormatCurrency, etc, actually call into the VBA Format$ code.

 

 

To answer the question, no.  The VBA runtime is freely redistributable, but it is also **large**.  Back in 1996 we did not want to force people to download the large VBA runtime.  IE was on our case for every kilobyte we added to the IE download.

 

 

However, FormatNumber, etc, were written by the same developer who implemented Format$ in VBA.  That was the legendary Tim Paterson, who you may recall wrote a little program called QDOS that was eventually purchased by Microsoft and called MS-DOS.  And let me tell you, when I was an intern in 1993 I had to debug Format$ -- there are **very good reasons** why we decided to go with a stripped-down let's-solve-the-90%-usage-case version for VBScript\!  Format$ is one of the most enormously complicated hard-core-bit-twiddling functions that I've ever seen.  That's what happens to these "everything but the kitchen-sink" functions that tries to be all things to all callers -- they get enormously bloated and complex.  Porting that thing to a new codebase and maintaining it independently would have been nightmarish.

 

 

Here's a comment that Tim wrote into the FormatNumber code on September 16th, 1996 which confirms the reader's observation:

 

 

       // We have more digits that we're not using.    Round if needed.

       // This is simplistic rounding that does not use IEEE

       // rules (rounding even if exactly x.5).    This simple

       // way is how VBA Format$ works, so it's good enough for us.

