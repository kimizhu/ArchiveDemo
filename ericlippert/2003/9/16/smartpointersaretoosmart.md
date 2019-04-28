# Smart Pointers Are Too Smart

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 5:05:00 PM

-----

Joel's law of leaky abstractions rears its ugly head once more.  I try to never use smart pointers because... I'm not smart enough.

 

 

COM programmers are of course intimately familiar with AddRef and Release.  The designers of COM decided to use reference counting as the mechanism for implementing storage management in COM, and decided to put the burden of implementing and calling these methods upon the users.

 

 

Now, it is very natural when using a language like C++ to say that perhaps we can encapsulate these semantics into an object, and let the C++ compiler worry about calling the AddRefs and Releases in the appropriate constructors, copy constructors and destructors.  It is very natural and very tempting, and I avoid template libraries that do so like the plague. 

 

 

Everything usually works fine until there is some weird, unforeseen interaction between the various parts.  Let me give you an example:

 

 

Suppose you have the following situation: you have a C++ template library map mapping IFoo pointers onto IBar pointers, where the pointers to the IFoo and IBar are in smart pointers.  You want the map to take ownership of the pointers.  Does this code look correct?

 

 

map\[srpFoo.Disown()\] = srpBar.Disown(); 

 

 

It sure looks correct, doesn't it?

 

 

Look again.  Is there a memory leak there?   

 

 

I found code like this in a library I was maintaining once, and since I had never used smart pointer templates before, I decided to look at what exactly this was doing at the assembly level. A glance at the generated assembly shows that the order of operations is:

 

 

1\) call srpFoo.Disown() to get the IFoo\* 

2\) call srpBar.Disown() to get the IBar\* 

3\) call map's operator\[\] passing in the IFoo\* , returning an IBar\*\* 

4\) do the assignment of the IBar\* to the address returned in (3).

  

So where is the leak? This library had C++ exception handling for out-of-memory exceptions turned on.  *If the* operator\[\]* throws an out-of-memory exception then the not-smart  *IFoo\** and *IBar\** presently on the argument stack are both going to leak. *  

 

 

The correct code is to copy the pointers before you disown them:

 

 

map\[srpFoo\] = srpBar;

srpFoo.Disown();

srpBar.Disown();

 

 

Before the day that I found this I had *never* been in a situation before where I had to think about the C++ order of operations for assigning and subscripting in order to get the error handling right\!  *The fact that you have to know these picky details about C++ operator semantics in order to get the error handling right is an indication that people are going to get it wrong.* 

 

 

Let me give you another example.  One day I was adding a feature to this same library, and I noticed that I had caused a memory leak.  Clearly I must have forgotten to call Release somewhere, or perhaps some smart pointer code was screwed up.  I figured I'd just put a breakpoint on the AddRef and Release for the object, and figure out who was calling the extra AddRef.

 

 

Here -- and I am not making this up\! -- is the call stack at the point of the first AddRef:

 

 

ATL::CComPolyObject\<CLayMgr\>::AddRef

ATL::CComObjectRootBase::OuterAddRef

ATL::CComContainedObject\<CLayMgr\>::AddRef

ATL::AtlInternalQueryInterface

ATL::CComObjectRootBase::InternalQueryInterface

CLayMgr::\_InternalQueryInterface

ATL::CComPolyObject\<CLayMgr\>::QueryInterface

ATL::CComObjectRootBase::OuterQueryInterface

ATL::CComContainedObject\<CLayMgr\>::QueryInterface

CDDS::FinalConstruct

ATL::CComPolyObject\<CDDS\>::FinalConstruct

ATL::CComCreator\<ATL::CComPolyObject\<CDDS\> \>::CreateInstance

CTryAssertComCreator\<ATL::CComPolyObject\<CDDS\> \>::CreateInstance

ATL::CComClassFactory2\<CDLic\>::CreateInstance(ATL::CComClassFactory2

CTryAssertClassFactory2\<CDLic\>::CreateInstance(CTryAssertClassFactory2\<CDLic\>

 

 

Good heavens\!   

 

 

Now, maybe you ATL programmers out there are smarter than me.  In fact, I am almost certain you are, because I have not the faintest idea what the differences between a CComContainedObject, a CComObjectRootBase and a CComPolyObject are\! It took me **hours** to debug this memory leak.  So much for smart pointers saving time\!

 

 

I am too dumb to understand that stuff, so when I write COM code, my implementation of AddRef is **one line long**, not hundreds of lines of dense macro-ridden, templatized cruft winding its way through half a dozen wrapper classes.

