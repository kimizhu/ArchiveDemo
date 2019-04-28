# SimpleScript Part Two: Class Factories Are Also Boring

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2004 9:51:00 PM

-----

Before I get into it, a Lambda poster pointed me at the [NullScript](http://www.iwebthereforeiam.com/projects/NullScript.asp) project, which is a very interesting illustration of how reverse engineering works.  It's an implementation of a "null" script engine -- an engine with no language -- in ATL, which the intrepid developer created in order to try and understand how ASP works.  Typical programmer\! They could have just [asked me,](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53046.aspx) but where's the fun in that?  J  

I want to go well beyond the scope of the NullScript project, in several ways.  

First off, someone asked *why not use ATL*?  Aside from my general distaste for the ATL style of programming, the more fundamental reason is that ATL is all about **hiding** the details of how COM works from you.  One of the points of this exercise is to show **how** it works.  In ATL, you see 

    COM\_INTERFACE\_ENTRY(IDispatch)

And what does that tell you?  It's just a black box, and when you open it up, it's full of weird macros that I don't understand.  I'd much rather show you guys how this works at a **much less abstract** level.

Second, the point of NullScript was to have the smallest feature set that still worked, because it was being used as a probing tool, not a language tool.  I want to actually talk about practical concerns here, like good language design, how to implement IDispatch correctly, etc, not just write a logging tool.

Today, more boilerplate code.  I've added a class factory. 

For those of you who don't know much about COM internals, you might wonder how instantiating a COM object works.  Basically, it goes like this.

First, there's a progid -- the human-readable string that describes the object you want to create.  As I mentioned yesterday, we create registry keys for the progid that map it to a class id…

HKCRSimpleScriptCLSID(Default) = "{...}"

… and then map that class id to a DLL…

HKCRCLSID{...}InprocServer32(Default) = "c:simplescript.dll"

Thus, the registry has enough information to determine everything you need to know to actually get the code in memory -- the location of the code and the unique identifier for the class.

To create the object, COM loads up the DLL and calls DllGetClassObject in [dllmain.cpp](http://blogs.msdn.com/ericlippert/articles/105188.aspx).  It doesn't ask for an object instance, as you might expect.  Instead, it asks for a class factory -- an object that creates objects.  The class factory in [classfac.cpp](http://blogs.msdn.com/ericlippert/articles/105858.aspx) then knows how to create the actual object.  Why this indirection?  Because  DllGetClassObject is *fundamentally not extensible* but COM objects are extensible -- you can add any interface on that you like -- thus the convention is to create a COM object that does the creation work rather than put more smarts into the entrypoint.  For example, the object might implement [IClassFactory2](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/htm/cmi_c_641e.asp), which enables licensing semantics.  

If you take a look at the implementation, you'll see that we have your basic class factory going here.  There's nothing fancy.

You might wonder what the locking mechanism is for --  this is for the scenario where you know that you're going to be creating a whole bunch of objects and need to ensure that you're not unloading the DLL unnecessarily.  That's an extremely unlikely scenario for us, but I've implemented the code anyway because its cheap, easy and the right thing to do.

As you can see, all of the objects are thread safe so far.  We'll get into the threading model of the script engine in more detail later, but as a refresher, you might want to read [this](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx).

The class factory does everything but actually create the engine -- that's still E\_NOTIMPL.  

How far can we get now?  If we compile up the code, register the DLL and run it through WSH:

\<job\>  
\<script language="SimpleScript"\>  
Testing  
\</script\>  
\</job\>

Then we get the DLL loaded, the class factory created, and a call to create the engine, which fails:

Windows Script Host: An unimplemented function was called : SimpleScript

We're doing pretty well so far, but we're still a long way from computing 2 + 2 or writing "hello world".

Next time, I'll talk about the engine and site interfaces, engine state, and a skeleton of the engine interfaces, which I'll then flesh out over several entries.

