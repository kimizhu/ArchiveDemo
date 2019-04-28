# Why is it a bad idea to put script objects in Session scope?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 10:35:00 PM

-----

 

Often a web site will have a series of related pages requested one after the other by the same user.  As a convenience for the site developers, the ASP object model provides a Session object to store server-side state for a current user.  It also has a global "Application" object which stores state for an entire virtual root.   

 

 

Every so often some ASP coder out there tries to put a JScript or VBScript object into the Session (or Application) object.  Things usually start going horribly wrong shortly thereafter -- either terrible performance ensues or things just break outright.

 

 

Well, like Groucho says, if it hurts when you do that, don't do that\!  Understanding why this is a bad idea will take us yet deeper into the land of threading models.

 

 

Marshaling 

 

 

I mentioned earlier that when you have an apartment threaded object, you need to be in the right apartment (thread) if you want to talk to its occupants (object instances).  But what if you are not?  What if you are running some code in thread Alpha that *really* needs to call a method on an object running in thread Beta?  Fortunately, COM provides a mechanism called **cross-thread marshaling** to achieve this.   

 

 

The behind-the-scenes details are not particularly important to our discussion; suffice to say that windows messages are involved.  The important thing to know is that when you marshal a call across threads **the calling thread pauses until the called thread responds**.  That seems reasonable -- after all, when you call an ordinary function you sort of "pause" until the function returns.  

 

But threads are usually busy doing *something*.  If the called thread is not responding to messages because it is busy doing work of its own then the calling thread waits, and waits, and waits...

 

 

To continue with our previous apartment threading analogy, it is rather like each apartment has a mailbox.  If you're in apartment Beta and you need someone in apartment Alpha to do something for you, you write up a request and hand it to the mailman who in turn sticks it in Alpha's mailslot.  Alpha's occupants might be busy doing work and ignoring their mail, or they may have a huge stack of mail to get through, or they might get right on it.  You, over in apartment Beta, can't do anything but wait for the mailman to deliver their reply.

 

 

 

And of course, even if the callee thread is completely responsive and ready to service your request, obviously calling across threads is orders of magnitude more expensive than calling in-thread.  An in-thread call requires some arguments to be put on the stack and maybe stash a few values in registers.  A cross-thread call gets the operating system involved in a comparatively deep and complex way.

 

Performance Woes

 

 

Now you have enough information to figure out why putting script objects in Session scope is a bad idea as far as performance is concerned.  

 

Each ASP page is running on its own thread from the thread pool.  The thread that reads the object from session state will likely not be the thread that put the object there, but the script object is apartment threaded.  That means that essentially any page that accesses that session object must wait its turn for all other pages using that session object to finish up, because **the marshaling code blocks the calling thread until the callee is available.**** ** You end up with a whole lot of blocking threads, and blocking threads are not fast.   

 

 

Application scope is even worse -- if you put a script object in Application scope then **every page in the entire vroot that accesses the Application object must wait its turn** for the original thread to be free.  You've effectively single-threaded your web server.

 

 

Script Engine Of The Living Dead

 

 

But it gets worse.  Remember, when the page is done being served up, the engine is torn down.  The compiled state remains, but the runtime state is thrown away.  So suppose you have a JScript object sitting in the Session object, and the page that put it there was destroyed a few microseconds ago and the engine put back into the engine pool.  Now on another page in the same session you try to fetch the JScript object and call a method on it.  Where's the runtime state associated with that object?  It's gone, dude.   

 

 

When the script engine is torn down after the original page is served, the teardown code detects that there are existing objects that are owned by external callers.  The script engine can't destroy these objects, otherwise the caller would crash when the caller tried to destroy them later.  That's a fundamental rule of COM -- you can't destroy an object to which someone holds a reference.  

 

But we know that the object is going to be useless, so what we do is tell the object "the engine you require for life is going away.  Throw away everything you own and become a zombie."   

 

 

These zombie objects look like script objects, but when you actually try to do something to them -- call a method, fetch a property, whatever -- they don't actually do anything.  They can't -- all the infrastructure they need to live is gone, but they can't die.  Basically they wander the night in ghostly torment until they are freed by whatever code is holding the reference.

 

 

So not only are those script objects sitting in Session state wrecking your performance, they're not even *useful* for anything once the original page goes away.

 

 

Incidentally, Windows Script Components each have their own engine which stays alive as long as they do, so WSC's are not affected by the zombie issue.  They are still apartment threaded though.

 

 

Arrays Are Almost As Bad

 

 

JScript arrays are objects, so everything said above applies to them.  VBScript arrays are not objects (more on the differences between these two kinds of arrays later) but even still, you shouldn't put them in Session scope either.  Though they do not suffer from the threading problems or the lifetime problems mentioned above, arrays are stored into and passed out of Session scope using our old friend copy-in-copy-out semantics.  That means that **every single time **you index into an array in Session scope, **a copy of the array is made first**.  If that array is big, that's a big performance problem.  

 

Why do we do copy-in-copy-out?  *Because* arrays are not marshaled\!  We return to the fundamental problem: what if two pages tried to read and write the array in the Session object at the same time?  The memory could be corrupted.  We really don't have any good way to synchronize access to the array, so instead we simply make a complete copy of it every time someone tries to read or write it.  This is extremely expensive, but it keeps the server heap from being corrupted.  A corrupted web server heap can ruin your whole day.

 

 

My advice is to not even go there.  **Don’t put information into Session scope unless you absolutely have to.**** ** If you must, put in strings.  There are lots of ways to store arrays or objects as strings and reconstitute them as needed.

