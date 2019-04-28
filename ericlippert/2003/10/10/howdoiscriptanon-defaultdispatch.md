# How Do I Script A Non-Default Dispatch?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2003 2:43:00 PM

-----

 

 

As I've discussed previously, the script engines always talk to objects on the late-bound IDispatch interface.  The point of this interface is to allow a script language to call a method or reference a property of an object by giving the name of the field and the arguments.  When the dispatch object is invoked, the object does the work of figuring out which method to call on which interface.

 

 

But what if an object supports two interfaces IFoo and IBar both of which you want to be able to call late-bound?  The most common way to solve this problem is to create two late-bound interfaces, IFooDisp and IBarDisp.  So when you ask the object for IFooDisp, you get an interface that can do late-bound invocation on IFoo, and similarly for IBarDisp.

 

 

What happens when you ask the object for an IDispatch interface?  It's got to pick one of them\!  The one it picks is called the "default dispatch".

 

 

In JScript and VBScript, when you create an object (via new ActiveXObject in JScript or CreateObject in VBScript) the creation code always returns the default dispatch.  Furthermore, in JScript, when you fetch a property on an object and it returns a dispatch object, we ask the object to give us the default dispatch.  So in JScript, there is no way to script a non-default dispatch.

 

 

What about in VBScript?  There's an irksome story here, again featuring a really bad mistake made by yours truly.  At least I *meant* well.

 

 

I didn't write the variant import code, and I had always assumed that VBScript did the same thing as JScript -- when an object enters the script engine from outside, we query it for its default dispatch.  As it turns out, that's not true.  In VBScript, for whatever reason, we just pass imported dispatch objects right through and call them on whatever dispatch interface the callee gives us.

 

 

One day long ago we found a security hole in IE.  The details of the hole are not important -- but what's interesting about it was the number of things that had to go wrong to make the hole an actual vulnerability.  Basically the problem was that one of the built-in IE objects had two dispatch interfaces, one designed to be used from script and one for internal purposes.  The for-script dispatch interface was the default, and it was designed to participate in the IE security model.  The internal-only interface did not enforce the IE security model, and in fact, there was a way to use the object to extract the contents of a local disk file and send it out to the internet, which is obviously badness. Furthermore, there was a way to make *another* object return the non-default interface of the broken object.   

 

 

JScript did not expose this vulnerability because it always uses the default dispatch even when given a non-default dispatch.  But the vulnerability *was* exposed by VBScript.  All these flaws had to work together to produce one vulnerability.  Now, when you find a security hole that consists of multiple small flaws, the way you fix it is **not** to just **patch one thing and hope for the best**.  You **patch everything you possibly can**.  Remember, secure software has **defense in depth**.  Make the attackers have to do *twelve* impossible things, not *one* impossible thing, because sometimes you're wrong about what's impossible.   

 

 

Hence, when we fixed this hole, we fixed *everything*.  We made sure that the object's persistence code could no longer read files off the local disk.  In case there was still a way to make it read the disk that the patch missed, we fixed the object model so that it never returned a non-default interface to a script.  In case there was still a way to do both those things that we missed, **we also turned off VBScript's ability to use non-default dispatches.**

 

 

That last one turned out to be a huge mistake.  The fact that **I had believed that VBScript always talked to the default dispatch** does not logically imply that every **VBScript user read my mind and knew that successfully using a non-default dispatch was some sort of fluke**.  People naturally assume that if they write a program and it works, then it works because the language designers wanted them to be able to write that program\!

 

 

As it turned out, there were *plenty* of programs out there in the wild that used VBScript to script non-default dispatch interfaces returned by method calls, and I broke *all* of them through trying to make IE safer.  We ended up shipping out a new build of VBScript with the default dispatch feature turned back on a couple of days later.  (Of course all the fixes to IE were sufficient to mitigate the vulnerability on their own, and those were not changed back.)

 

 

The morals of the story are:

 

 

1\)      If you want to use a non-default dispatch, you have to use VBScript.

2\)      There is no way to make VBScript give you a specific non-default dispatch.  In VBScript, once you have a dispatch, that's the one you're stuck with.

3\)      Defense in depth is a good idea, but it's not such a good idea to go so deep that backwards compatibility is broken if you can avoid it\!

