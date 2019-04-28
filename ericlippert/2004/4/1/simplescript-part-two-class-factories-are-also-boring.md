<div id="page">

# SimpleScript Part Two: Class Factories Are Also Boring

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2004 9:51:00 PM

-----

<div id="content">

<div>

<span>Before I get into it, a Lambda poster pointed me at the [NullScript](http://www.iwebthereforeiam.com/projects/NullScript.asp) project, which is a very interesting illustration of how reverse engineering works.  It's an implementation of a "null" script engine -- an engine with no language -- in ATL, which the intrepid developer created in order to try and understand how ASP works.  Typical programmer\! They could have just [asked me,](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53046.aspx) but where's the fun in that?  </span><span>J</span><span>  </span>

<span>I want to go well beyond the scope of the NullScript project, in several ways.  </span>

<span>First off, someone asked *why not use ATL*?  Aside from my general distaste for the ATL style of programming, the more fundamental reason is that ATL is all about **<span>hiding</span>** the details of how COM works from you.  One of the points of this exercise is to show **<span>how</span>** it works.  In ATL, you see </span>

<span>    COM\_INTERFACE\_ENTRY(IDispatch)</span>

<span>And what does that tell you?  It's just a black box, and when you open it up, it's full of weird macros that I don't understand.  I'd much rather show you guys how this works at a **<span>much less abstract</span>** level.</span>

<span>Second, the point of NullScript was to have the smallest feature set that still worked, because it was being used as a probing tool, not a language tool.  I want to actually talk about practical concerns here, like good language design, how to implement IDispatch correctly, etc, not just write a logging tool.</span>

<span>Today, more boilerplate code.  I've added a class factory. </span>

<span>For those of you who don't know much about COM internals, you might wonder how instantiating a COM object works.  Basically, it goes like this.</span>

<span>First, there's a progid -- the human-readable string that describes the object you want to create.  As I mentioned yesterday, we create registry keys for the progid that map it to a class id…</span>

<span>HKCRSimpleScriptCLSID(Default) = "{...}"</span><span></span>

<span>… and then map that class id to a DLL…</span>

<span>HKCRCLSID{...}InprocServer32(Default) = "c:simplescript.dll"</span><span></span>

<span>Thus, the registry has enough information to determine everything you need to know to actually get the code in memory -- the location of the code and the unique identifier for the class.</span>

<span>To create the object, COM loads up the DLL and calls </span><span>DllGetClassObject</span><span> in [dllmain.cpp](http://blogs.msdn.com/ericlippert/articles/105188.aspx).  It doesn't ask for an object instance, as you might expect.  Instead, it asks for a class factory -- an object that creates objects.  The class factory in [classfac.cpp](http://blogs.msdn.com/ericlippert/articles/105858.aspx) then knows how to create the actual object.  Why this indirection?  Because  </span><span>DllGetClassObject</span><span> is *<span>fundamentally not extensible</span>* but COM objects are extensible -- you can add any interface on that you like -- thus the convention is to create a COM object that does the creation work rather than put more smarts into the entrypoint.  For example, the object might implement [IClassFactory2](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/htm/cmi_c_641e.asp), which enables licensing semantics.  </span>

<span>If you take a look at the implementation, you'll see that we have your basic class factory going here.  There's nothing fancy.</span>

<span>You might wonder what the locking mechanism is for --  this is for the scenario where you know that you're going to be creating a whole bunch of objects and need to ensure that you're not unloading the DLL unnecessarily.  That's an extremely unlikely scenario for us, but I've implemented the code anyway because its cheap, easy and the right thing to do.</span>

<span>As you can see, all of the objects are thread safe so far.  We'll get into the threading model of the script engine in more detail later, but as a refresher, you might want to read [this](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx).</span>

<span>The class factory does everything but actually create the engine -- that's still </span><span>E\_NOTIMPL</span><span>.  </span>

<span>How far can we get now?  If we compile up the code, register the DLL and run it through WSH:</span>

<span>\<job\>  
</span><span>\<script language="SimpleScript"\>  
</span><span>Testing  
</span><span>\</script\>  
</span><span>\</job\></span>

<span>Then we get the DLL loaded, the class factory created, and a call to create the engine, which fails:</span>

<span>Windows Script Host: An unimplemented function was called : SimpleScript</span>

<span>We're doing pretty well so far, but we're still a long way from computing 2 + 2 or writing "hello world".</span>

<span>Next time, I'll talk about the engine and site interfaces, engine state, and a skeleton of the engine interfaces, which I'll then flesh out over several entries.</span>

</div>

</div>

</div>

