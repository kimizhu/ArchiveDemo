# Reading Code Is Hard, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/15/2004 11:05:00 AM

-----

I was thinking about [this](http://blogs.msdn.com/ericlippert/archive/2004/06/14/155316.aspx)a bit more and talking with [Larry Osterman](http://weblogs.asp.net/larryosterman)yesterday, and I came up with some perhaps slightly more germane tips on how to read and debug code that you didn't write. 

First, it is highly likely that the machine code you're debugging will not match the source code -- either because you've simply got outdated sources, or because the optimizations have been so severe as to swiss-cheese the relationship between the source and the binary, or because you've got bad symbols or whatever.  I am certainly not an assembly programmer, but I know enough assembly language and enough about calling conventions that I can at least make a stab at debugging code where the sources and the binaries aren't quite in sync.  Even knowing things like "odds are good that the 'this' pointer is in ECX" can go a long way towards making a live debug session more productive. 

Second, you remember that *Calvin and Hobbes* cartoon where Calvin sneaks up on Hobbes, scares him, and then learns that scaring a sleeping tiger is a bad idea?  Calvin drags himself off muttering **"I have got to start listening to those quiet, nagging doubts."** 

That cartoon changed my life.  I realized just how often I had made a mistake which in retrospect, I knew ahead of time was probably going to be a mistake, but for whatever reason, I ignored the quiet, nagging doubt and forged boldly ahead.  I decided that I did not want my tombstone to say “He died from something stupid and completely avoidable.“ 

Trust your brain.  Yes, hunches can be wrong, but when they are, it's usually pretty cheap to find out that your hunch is wrong.  And if it's right, you can save a lot of time.  When I'm debugging code that I didn't know, I try to listen to those quiet, nagging doubts.  The memory window just made a byte flash red, indicating that it just changed.  Hmm.  Was I expecting memory to change in that function call?  Did owned memory change, or was that a heap corruption?  Hmm, this local variable suddenly changed value -- is this a symbol problem?  Is the variable shadowed, and I just stepped out of an inner scope?  Chase stuff like that down. 

Just recently I found an extremely obscure bug in Excel by following a hunch -- I noticed that there was a place where a heap-allocated buffer was passed into a function but its length was not passed in.  Hmm.  Suspicious\!  Sure enough, once you trace all the logic through, some function ten levels deeper makes a bad assumption about the buffer size, and causes an error.  Fortunately the assumption is too conservative, not too permissive, so there is no buffer overrun bug there.  Also fortunately, the bug is extremely obscure -- normal users will not be running into it any time soon. 

Which brings me to my third suggestion.  Yesterday I said that you need to understand what the code does and why the developer made it do that.  But there's more -- **understand also how code changes over time.**  Clearly in this bug what must have happened is that the method originally took a small fixed-size buffer.  Someone modified the buffer allocation code to allocate a variable-sized buffer, but forgot that this one method deep in the code was written to assume a small fixed-size buffer.  Bugs often happen through minor errors in otherwise sensible refactoring.  Obviously no one would have written this particular bug into the code had they written the variable-sized-buffer code from scratch.  When you're reading a piece of code, **try to understand what the intended invariants are, and then think about ways that they could be violated**.  

Of course, knowing where those quiet, nagging doubts come from is hard, and perhaps that can't be taught.  It's a hard problem, no matter how you slice it.

