# SimpleScript, Part Zero

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/29/2004 10:40:00 AM

-----

I'm going to embark upon something ambitious here.  I got an email from Mike, our friendly support engineer on the east coast with the subject line "**Need lots of info**".  Apparently he was contacted by a customer who wants to implement a **new script engine** that **supports debugging**, and, here's the part that you've got to really grit your teeth over: **in C\#.**  

My initial response was, of course, *"They're crazy -- talk them out of it\!"*  Not to put too fine a point on it, that's an immense amount of work in any language.  But C\#? The script interfaces were designed long, long before there was managed code.  It's quite likely that it's going to be very hard to write a straightforward interop layer for those interfaces.  "Could they use the managed IVsa interfaces instead?" I asked Mike. 

I haven't heard back yet, but I got to thinking last night that maybe it would be interesting to develop a script engine from scratch, describing every interface as I implement it, giving some of the design history behind the interfaces, etc, etc, etc.  I could start it in C++ and then see whether there were ways to make it work well in managed code, etc. 

I could then blog a little bit of the code every few days.  This would keep me in blog topics for weeks, certainly.  Probably months.  All of this stuff happens in my spare time, and I have precious little of that, particularly given the number of bugs I have to fix before the Whidbey beta.  But I'll see what I can do. 

I'm not sure that a piecemeal daily blog approach is even the right way to handle this kind of material, but I'm willing to make the experiment.  

This is probably how it will go: 

Phase One: Basic Infrastructure  
\-       dll entrypoints  
\-       registration  
\-       class factory  
\-       engine skeleton 

Phase Two: Building the engine  
\-       script engines as state machines  
\-       threading concerns  
\-       named items  
\-       event handling 

Phase Three: Processing the language  
\-       grammar  
\-       lexer  
\-       parser  
\-       code generator  
\-       interpreter  
\-       runtime library 

Phase Four: Debugger support  
\-       breakpoints  
\-       evaluation  
\-       syntax colouring  
\-       context 

Phase Five: Managed Code  
\-       ? 

That's just off the top of my head.  We'll probably mix it up a bit as we go.

