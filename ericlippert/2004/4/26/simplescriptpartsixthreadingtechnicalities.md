# SimpleScript Part Six: Threading Technicalities

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/26/2004 8:56:00 PM

-----

 

Refresher Course 

Before you read this, you might want to take a quick refresher on my [original posting](/ericlippert/archive/2003/09/18/53041.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx") on the script engine threading model.  That was a somewhat simplified (\!) description of the actual script engine contract.  Let me just sum up: 

  - **free threaded objects** can be called from any thread at any time; the object is responsible for synchronizing access to shared resources
  - **apartment threaded objects** can have multiple instances on multiple threads but once an object is on a thread, it can only be called from that thread.  The caller is responsible for always calling an object on the thread on which it was created.  The object is responsible for synchronizing access to resources shared across instances.
  - **rental threaded objects** can be called on any thread, but the caller is responsible for ensuring that the object is only called from one thread at a time.
  - an **initialized** engine is apartment threaded, with a couple exceptions -- InterruptScriptThread and Clone can both be called from any thread on an initialized engine.
  - an **uninitialized** engine is free-threaded

Things Get More Complicated 

That's actually not quite right.  It would be more accurate to say that **an** **uninitialized engine is rental threaded**.  Why?  Because otherwise it would be legal to do really dumb things like call Close on two different threads at the same time.  If you look carefully at the code, you'll see that most of the methods are not robust in the face of full-on multithreading.  It's the "the host isn't a bozo" threading model\! 

This is a ridiculously complex threading contract, I know.  From COM's point of view, the script engine is free threaded -- the restrictions are so arcane that clearly the engine has got to be the one enforcing the rules, not COM, which is why you'll see lots of calls to check that the caller is on the right thread in my code. 

If you take a look at the registration code, you'll see that I register the script engine as "Both", which means "either free threaded or apartment threaded, we'll sort it out".  What the heck is up with that?  Why not just call it "free threaded" and be done with it? 

Because again, I oversimplified the description of an apartment in my original posting.  A single-threaded-apartment (STA) object is created on a thread and is always called on that thread.  Think of a person (an object) in a room (a thread) -- you want to talk to them (call a method), you go to their room.  You can put as many people in a room as you'd like, and build as many rooms as you'd like, but you want to call a method, you do it from the thread where the callee lives. That's all fine and good. 

But there is also a multi-threaded apartment\!  Imagine that you take some of the rooms and you knock holes in the walls.  If you're in one room and you want to talk to someone in another room, you don't have to go there, you can just yell through the hole.  The guy listening to you is responsible for synchronizing all the shouting going on, but at least he knows that no one is going to be trying to talk through a wall with no hole in it.  

In any given process there is one "main" STA, possibly many more secondary STAs, and one MTA.  You can't have two distinct sets of rooms that mutually communicate but don't talk to each other. 

I briefly described in an [earlier post](http://weblogs.asp.net/ericlippert/archive/2003/09/18/53050.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/18/53050.aspx") the way that COM marshals calls "through the walls" from one apartment to another.  Well, if we register the script engine as a free threaded object, COM is going to think that it lives in the default MTA, and that any STA objects created by the process live in their own STAs, and therefore, all calls between the two apartments are going to have to be marshaled by the Free Threaded Marshaler.  That extra indirection **really screws up your performance numbers**, lemme tell ya.  We register as "both" threaded, and COM says "*OK, you sort it out then if you're so smart*", which we do by requiring that the host call us on the right thread and give us objects that are on the right thread.  

Even that is still a considerable oversimplification -- there's still the Neutral Threaded Apartment that I haven't talked about yet, and the interactions between the CLR threading model and the COM threading model, and how marshaling really works, but that's getting out of my depth.  Go ask [Christopher Brumme](/cbrumme "http://blogs.msdn.com/cbrumme") if you've got questions about that stuff. 

Honouring Our End Of The Script Engine Contract 

Let's take a look at the script engine contract through the example of one of its objects that we've already implemented -- the named item list.  There are only four operations that can be performed on the named item list, and we know when they can be performed according to our contract and implementation: 

  - Add can only be called on an **initialized** engine, and hence only **from the engine thread**.  Add **writes** to the named item list.
  - Reset can only be called on an **initialized** engine as it is being moved to **uninitialized** state, hence only from the engine thread.  Reset **writes** to the named item list by removing non-persisting entries.
  - Clear is only called when the engine is going to **closed** state.  The engine might already be in **uninitialized** state and hence, **this can be called on any thread if the engine is uninitialized**.  However if the engine is **initialized** then it can only be called from the **main engine thread**.  Clear **writes** to the named item list by removing all entries.
  - Clone can be called on any thread, and **reads** from the named item list.

What then are the possible threading conflicts?  The three writers, Add, Reset and Clear cannot be called at the same time on different threads by virtue of the fact that the engine must be initialized if Add or Reset are being called.  There are a few cases which for completeness we should get right, but are in reality extremely unlikely.  Why would any host be so dumb as to Clone an engine while in the middle of a call to Add?  Or worse, Clone during Close?  I won't assume that hosts won't pull shens like that, even though they are very unlikely. 

What about two Clones at the same time on two different threads?  On the one hand, they're only reading, so why should they block?  On the other hand, boy, do I ever not want to implement single-writer-multi-reader mutexes just to make that extremely unlikely case marginally faster.  

Therefore, we'll do it the easy way.  The only thing we *really* need to worry about practically is one thread doing a Clone while another thread is doing a Reset, but we'll get it right for all the cases.  The first thing we'll do when we enter any of those methods is enter a critical section, and the last thing we'll do before we leave is exit it.  Rather than mess around with the operating system's somewhat gross critical section code in the object itself, I define a handy object to wrap it.  See [mutex.cpp](http://weblogs.asp.net/ericlippert/articles/116364.aspx "http://weblogs.asp.net/ericlippert/articles/116364.aspx") for the implementation. 

Something to note about this implementation is that it uses [InitializeCriticalSectionAndSpinCount](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsectionandspincount.asp "http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsectionandspincount.asp") to initialize the critical section.  The comments there and [here](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsection.asp "http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsection.asp") are *required reading if you need to make critical sections work on heavily loaded **pre-Windows-XP** boxes*. Earlier versions **throw exceptions** rather than returning error codes, which means that every entry and exit to a critical section has to be protected with \_\_try blocks and you then have to [get the exception handling right](http://weblogs.asp.net/oldnewthing/archive/2004/04/22/118161.aspx "http://weblogs.asp.net/oldnewthing/archive/2004/04/22/118161.aspx")\!  The VBScript and JScript engines have all kinds of totally gross code in them to handle the edge case where a heavily loaded server runs out of memory just as a critical section is about to be entered.  (Yes, it happens. Every single out-of-memory case will eventually be exercised by a sufficiently loaded server, I know this from painful experience.) 

I'm going to skip all that totally gross code here and assume that we all live in the happy world of Windows XP, where the operating system actually returns sensible errors. 

I'm still trying to sort out how all this is going to work once code blocks are throw into the mix.  I'll try out a few things and see how it goes.  More bulletins as events warrant.

