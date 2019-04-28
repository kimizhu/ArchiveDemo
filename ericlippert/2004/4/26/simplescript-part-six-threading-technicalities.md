<div id="page">

# SimpleScript Part Six: Threading Technicalities

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/26/2004 8:56:00 PM

-----

<div id="content">

<span> </span>

<span>Refresher Course </span>

<span></span>

<span>Before you read this, you might want to take a quick refresher on my [original posting](/ericlippert/archive/2003/09/18/53041.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx") on the script engine threading model.  That was a somewhat simplified (\!) description of the actual script engine contract.  Let me just sum up: </span>

<span></span>

  - <span>**free threaded objects** can be called from any thread at any time; the object is responsible for synchronizing access to shared resources</span>
  - <span></span><span>**apartment threaded objects** can have multiple instances on multiple threads but once an object is on a thread, it can only be called from that thread.  The caller is responsible for always calling an object on the thread on which it was created.  The object is responsible for synchronizing access to resources shared across instances.</span>
  - <span></span><span>**rental threaded objects** can be called on any thread, but the caller is responsible for ensuring that the object is only called from one thread at a time.</span>
  - <span></span><span>an **initialized** engine is apartment threaded, with a couple exceptions -- </span><span>InterruptScriptThread</span><span> and </span><span>Clone</span><span> can both be called from any thread on an initialized engine.</span>
  - <span></span><span>an **uninitialized** engine is free-threaded</span>

<span></span>

<span>Things Get More Complicated </span>

<span></span>

<span>That's actually not quite right.  It would be more accurate to say that **<span>an</span>** **<span>uninitialized engine is rental threaded</span>**.  Why?  Because otherwise it would be legal to do really dumb things like call </span><span>Close</span><span> on two different threads at the same time.  If you look carefully at the code, you'll see that most of the methods are not robust in the face of full-on multithreading.  It's the "the host isn't a bozo" threading model\! </span>

<span></span>

<span>This is a ridiculously complex threading contract, I know.  From COM's point of view, the script engine is free threaded -- the restrictions are so arcane that clearly the engine has got to be the one enforcing the rules, not COM, which is why you'll see lots of calls to check that the caller is on the right thread in my code.</span><span> </span>

<span>If you take a look at the registration code, you'll see that I register the script engine as "Both", which means "either free threaded or apartment threaded, we'll sort it out".  What the heck is up with that?  Why not just call it "free threaded" and be done with it? </span>

<span></span>

<span>Because again, I oversimplified the description of an apartment in my original posting.  A single-threaded-apartment (STA) object is created on a thread and is always called on that thread.  Think of a person (an object) in a room (a thread) -- you want to talk to them (call a method), you go to their room.  You can put as many people in a room as you'd like, and build as many rooms as you'd like, but you want to call a method, you do it from the thread where the callee lives. That's all fine and good. </span>

<span></span>

<span>But there is also a multi-threaded apartment\!  Imagine that you take some of the rooms and you knock holes in the walls.  If you're in one room and you want to talk to someone in another room, you don't have to go there, you can just yell through the hole.  The guy listening to you is responsible for synchronizing all the shouting going on, but at least he knows that no one is going to be trying to talk through a wall with no hole in it.  </span>

<span>In any given process there is one "main" STA, possibly many more secondary STAs, and one MTA.  You can't have two distinct sets of rooms that mutually communicate but don't talk to each other. </span>

<span></span>

<span>I briefly described in an [earlier post](http://weblogs.asp.net/ericlippert/archive/2003/09/18/53050.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/18/53050.aspx") the way that COM marshals calls "through the walls" from one apartment to another.  Well, if we register the script engine as a free threaded object, COM is going to think that it lives in the default MTA, and that any STA objects created by the process live in their own STAs, and therefore, all calls between the two apartments are going to have to be marshaled by the Free Threaded Marshaler.  That extra indirection **really screws up your performance numbers**, lemme tell ya.  We register as "both" threaded, and COM says "*<span>OK, you sort it out then if you're so smart</span>*", which we do by requiring that the host call us on the right thread and give us objects that are on the right thread.  </span>

<span></span>

<span>Even that is still a considerable oversimplification -- there's still the Neutral Threaded Apartment that I haven't talked about yet, and the interactions between the CLR threading model and the COM threading model, and how marshaling really works, but that's getting out of my depth.  Go ask [Christopher Brumme](/cbrumme "http://blogs.msdn.com/cbrumme") if you've got questions about that stuff. </span>

<span></span>

<span>Honouring Our End Of The Script Engine Contract </span>

<span></span>

<span>Let's take a look at the script engine contract through the example of one of its objects that we've already implemented -- the named item list.  There are only four operations that can be performed on the named item list, and we know when they can be performed according to our contract and implementation: </span>

<span></span>

  - <span>Add</span><span> can only be called on an **<span>initialized</span>** engine, and hence only **<span>from the engine thread</span>**.  </span><span>Add</span><span> **<span>writes</span>** to the named item list.</span>
  - <span></span><span>Reset</span><span> can only be called on an **<span>initialized</span>** engine as it is being moved to **<span>uninitialized</span>** state, hence only from the engine thread.  </span><span>Reset</span><span> **<span>writes</span>** to the named item list by removing non-persisting entries.</span>
  - <span></span><span>Clear</span><span> is only called when the engine is going to **<span>closed</span>** state.  The engine might already be in **<span>uninitialized</span>** state and hence, **<span>this can be called on any thread if the engine is uninitialized</span>**.  However if the engine is **<span>initialized</span>** then it can only be called from the **<span>main engine thread</span>**.  </span><span>Clear</span><span> **<span>writes</span>** to the named item list by removing all entries.</span>
  - <span></span><span>Clone</span><span> can be called on any thread, and **<span>reads</span>** from the named item list.</span>

<span></span>

<span>What then are the possible threading conflicts?  The three writers, </span><span>Add</span><span>, </span><span>Reset</span><span> and </span><span>Clear</span><span> cannot be called at the same time on different threads by virtue of the fact that the engine must be initialized if </span><span>Add</span><span> or </span><span>Reset</span><span> are being called.  There are a few cases which for completeness we should get right, but are in reality extremely unlikely.  Why would any host be so dumb as to </span><span>Clone</span><span> an engine while in the middle of a call to </span><span>Add</span><span>?  Or worse, </span><span>Clone</span><span> during </span><span>Close</span><span>?  I won't assume that hosts won't pull shens like that, even though they are very unlikely. </span>

<span></span>

<span>What about two </span><span>Clones</span><span> at the same time on two different threads?  On the one hand, they're only reading, so why should they block?  On the other hand, boy, do I ever not want to implement single-writer-multi-reader mutexes just to make that extremely unlikely case marginally faster.  </span>

<span></span>

<span>Therefore, we'll do it the easy way.  The only thing we *<span>really</span>* need to worry about practically is one thread doing a </span><span>Clone</span><span> while another thread is doing a </span><span>Reset</span><span>, but we'll get it right for all the cases.  The first thing we'll do when we enter any of those methods is enter a critical section, and the last thing we'll do before we leave is exit it.  Rather than mess around with the operating system's somewhat gross critical section code in the object itself, I define a handy object to wrap it.  See [mutex.cpp](http://weblogs.asp.net/ericlippert/articles/116364.aspx "http://weblogs.asp.net/ericlippert/articles/116364.aspx") for the implementation. </span>

<span></span>

<span>Something to note about this implementation is that it uses </span><span>[InitializeCriticalSectionAndSpinCount](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsectionandspincount.asp "http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsectionandspincount.asp")</span><span> to initialize the critical section.  The comments there and [here](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsection.asp "http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/initializecriticalsection.asp") are *required reading if you need to make critical sections work on heavily loaded **<span>pre-Windows-XP</span>** boxes*. Earlier versions **<span>throw exceptions</span>** rather than returning error codes, which means that every entry and exit to a critical section has to be protected with </span><span>\_\_try</span><span> blocks and you then have to [get the exception handling right](http://weblogs.asp.net/oldnewthing/archive/2004/04/22/118161.aspx "http://weblogs.asp.net/oldnewthing/archive/2004/04/22/118161.aspx")\!  The VBScript and JScript engines have all kinds of totally gross code in them to handle the edge case where a heavily loaded server runs out of memory just as a critical section is about to be entered.  (Yes, it happens. Every single out-of-memory case will eventually be exercised by a sufficiently loaded server, I know this from painful experience.) </span>

<span>I'm going to skip all that totally gross code here and assume that we all live in the happy world of Windows XP, where the operating system actually returns sensible errors. </span>

<span></span>

<span>I'm still trying to sort out how all this is going to work once code blocks are throw into the mix.  I'll try out a few things and see how it goes.  More bulletins as events warrant.</span>

</div>

</div>

