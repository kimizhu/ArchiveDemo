# SimpleScript Part One: DllMain is Boring

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2004 11:01:00 PM

-----

In talking with our support engineer it's just become more muddled.  I'm pretty sure now actually that the customer does not want to build a script engine, but whether they want to build a script editor, a script host or a script debugger is unclear.

Before I go on, let me take this opportunity to say that anyone who wants to create a script debugger from scratch really *is* crazy.  It took us *many* programmer-years to implement the Microsoft Script Debugger.  There are dozens of interfaces that you need to get right.  A debugger, even a simple, basic one like the MSD is a very complex piece of software.  *I* don't understand how the MSD works, and therefore I'm not about to try and explain it to anyone else\!  Unless you have a lot of experience developing debugging technologies and understand it inside out, I'd recommend against going anywhere near the script debugger interfaces from the debugger side.  From the engine and host sides, maybe, but not the debugger side.

I'm going to forge boldly ahead with my plan to develop a script engine from scratch.  And if that works out, maybe we'll do a script host as well. This will take some time, be warned.

I'm going to post the code so far as "articles" so that they don't show up in the feed, and then call out any "interesting" parts of the code as posts.  I've just dropped in the first few files.  So far we can register and unregister the script engine, but not actually create the engine yet.

The code should compile against VC6 or above, and I'm only testing it on Windows XP.  

I don't normally like to get all legal, but I think this would be a good time to point out that standard disclaimers apply.  This code is provided as a public service.  I'm writing it from scratch in my spare time, and I don't have a phalanx of Microsoft testers ensuring that everything works.  Code is provided as-is, with no warranties expressed or implied, if you compile it and run it and the world ends, that's not my problem, etc, etc, etc.  Aside from that, you're free to do whatever you want to this code.

OK, now that we've got that out of the way, let's take a look at what's going on here.  So far this is just your basic Windows DLL.  This is pretty boring, but hey, we're just getting going here.

In [simplescript.def](http://blogs.msdn.com/ericlippert/articles/105179.aspx) we've got some boilerplate export code that says that this DLL exports the four standard functions to start up, register, unregister and shut down the DLL.

In [guids.h](http://blogs.msdn.com/ericlippert/articles/105182.aspx) I define some useful guids.  So far the only thing there is the class ID for our new engine, because we'll need that when we register it.  Since the guid is a structure, it must both be declared as a symbol in the header, and somewhere actually turned into object code.  The very short [guids.cpp](http://blogs.msdn.com/ericlippert/articles/105184.aspx) performs the latter task.

In [headers.h](http://blogs.msdn.com/ericlippert/articles/105181.aspx) we've got your straightforward "include the world" header file.  It might be wise to turn this into a precompiled header when this thing starts getting huge, but we'll cross that bridge when we come to it.

I am paranoid about putting assertions into my code.  The point of an assertion is to document in an active way what is known to be true about a program -- not what we hope is true, not what is true most of the time, but what must logically be true.  Assertions are better than comments because assertions will tell you if they are ever violated -- comments will not\!  In [assert.h](http://blogs.msdn.com/ericlippert/articles/105183.aspx) you'll see that I define a macro that determines whether a given condition is true, and some special-purpose macros that check for things like "is this memory valid?"  More on those in a minute.

I sense the outrage -- didn't [I just say that I hated macros](http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx)? Indeed I do, but in this case, I really like the ability to automatically determine the file and line number, and I want this stuff to be zero-impact in the retail build, so in this case they're worth it.  Notice also that I am using macros in a very specific way.  I'm not introducing new control flow primitives, etc.  No one is ever going to use this thing in an expression.  Also note that, where possible, the macro immediately calls a method which can be debugged.

If you take a look at those methods in [assert.cpp](http://blogs.msdn.com/ericlippert/articles/105186.aspx) you'll notice a few oddities.  First off, you'll notice that I do not call [IsBadWritePtr](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/memory/base/isbadwriteptr.asp) or [IsBadReadPtr](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/memory/base/isbadreadptr.asp), because these guys are incredibly gross.  Basically, the way that IsBadFooPtr works is that it simply attempts to do the action, wraps the attempt in a try-catch, and returns true if an exception is thrown.  If the memory really is bad because you're at the end of the stack then this can potentially fault the stack in a bad way.  If you're in a multi-threaded scenario then IsBadWritePtr can mess up your program due to race conditions.  

Now, I realize that in this case we're talking about some debug-only code here -- if either of these ever return true then there's a major bug that's got to be fixed\!  But still, they give me the shivers.  I would much rather have my assertions actually ask the operating system whether that's a good pointer than "try it an see" the way IsBadFooPtr does.  (And in fact there *is* some code in the script engines that needs to check whether a pointer is bad, and it is a main-line common scenario that it will be -- in a not-debug-only scenario like that you cannot rely on the IsBad methods.  Why we have to do that is another story, which I may blog about some time.)

And finally we come to the first bit of code that actually has semantic import -- the DLL startup, shutdown and registration code in [dllmain.cpp](http://blogs.msdn.com/ericlippert/articles/105188.aspx).

The shutdown code is straightforward -- as you will see in the next episode, we will maintain a reference count on the DLL and only shut it down when there are no outstanding references.  The startup code is even more straightforward -- we simply cache the module handle in a global variable.

It is **extremely important** that the startup and shutdown code be **boring, boring, boring**.  If you don't understand why that is, or you feel tempted to put something cool in there, read [this](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/dllmain.asp), [this](http://blogs.gotdotnet.com/cbrumme/permalink.aspx/dac5ba4a-f0c8-42bb-a5cf-097efb25d1a9), [this](http://weblogs.asp.net/oleglv/archive/2003/12/12/43068.aspx), [this](http://weblogs.asp.net/oleglv/archive/2003/12/12/43069.aspx), [this](http://weblogs.asp.net/oldnewthing/archive/2004/01/27/63401.aspx) and [this](http://weblogs.asp.net/oldnewthing/archive/2004/01/28/63880.aspx). 

The registration code is the only code that's interesting at all here, and even it is pretty boring.  First off, take a look at the coding style of DllRegisterServer.  Except in some specific exceptions, the code I write in this sample is going to be brain-dead obvious insofar as the error code paths and object lifetimes are concerned.  Almost every method I write will follow this pattern:

  - if internal method, assert arguments are good.  If public method, check arguments for validity.
  - Initialize everything that needs to be freed later to NULL.
  - Initialize out parameters to NULL.
  - Do stuff.  If you get an error, go to the bottom of the function
  - Fill in “out” parameters
  - Free everything 

Glancing through the code now, I see that there are a few places where I forgot to assert that input strings were valid.  I'll fix those up later.

Does this technique produce long, boring routines that emphasize the error handling and make it harder to see what the function is doing in the mainline case?  Yes.  But error handling and cleanup is sufficiently important that it is worth calling out -- I need to make sure that it is right, and the best way I know to do that is to make it obvious what everything is doing.  And besides, I'm going to keep the routines short enough that it should be fairly easy to see what they're doing even if every line of “main line“ code has three lines of error handling to go with it.

The registration code creates these keys:

HKCRSimpleScript(Default) = "SimpleScript Language Engine"  
HKCRSimpleScriptOLEScript  
HKCRSimpleScriptCLSID(Default) = "{...}"  
HKCRCLSID{...}(Default) = "SimpleScript Language Engine"  
HKCRCLSID{...}OLEScript   
HKCRCLSID{...}ProgID(Default) = "SimpleScript"  
HKCRCLSID{...}InprocServer32(Default) = "c:simplescript.dll"  
HKCRCLSID{...}InprocServer32ThreadingModel = "Both"  
  

as well as some category keys that say "this thing is a script engine".  The "OLEScript" tags are for backwards compatibility with VERY old script hosts, like in the Win16 timeframe -- strictly speaking I would imagine that they are no longer necessary, but it doesn't hurt.

The only thing interesting about the unregister code is that it **fails silently**.  Actually writing code that checks the error conditions, determines whether the keys couldn't be deleted because (a) they're not there, (b) they are there but you can't see them because you don't have access or (c) they are there but you don't have access to delete them is a pain in the rear.  It's not great that this fails silently when a non-admin tries to unregister the engine, but I'm not going to lose any sleep over it.

Next time, I'll build on this foundation a bit by adding a class factory and the skeleton of a script engine.  Then we can get into the actual script interfaces.

