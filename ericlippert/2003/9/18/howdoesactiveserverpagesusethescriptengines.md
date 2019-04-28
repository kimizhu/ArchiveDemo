# How does Active Server Pages use the script engines?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 5:32:00 PM

-----

It's always struck me as a little bit odd that Active Server Pages, a web server, encourages developers to use VBScript and JScript to write server-side scripts.  I mean, the whole point of a web server is that it produces complex strings (web pages are just strings of HTML after all) as blindingly fast as possible, on demand.  "Blindingly fast" and "script language" do not really go together, compared to, say, C. 

My opinions -- and you'd better believe I have plenty of strong ones\! -- on performance characteristics of the script engines will certainly come back in [future posts](http://blogs.msdn.com/ericlippert/category/2717.aspx?Show=All).  But for now I want to talk a little bit about what features we added to the script engines to make them work as well as they do for ASP. 

One of the ways you make a server fast is by caching everything you possibly can cache.  For example, suppose you need to do a job on a separate thread.  You'd probably create a thread, do some work on the thread, and throw it away when you're done.  OK, now suppose you need to do a million jobs but never more than, say, three at a time. Creating and destroying those million threads is potentially going to be a not-insignificant amount of the total processor time spent.  You'd be better off **creating three threads and re-using them as necessary**. 

This strategy is called “thread pooling”, and ASP certainly uses it.

(UPDATE: I've written a follow-up article on [Thread Happiness Disease](http://blogs.msdn.com/ericlippert/archive/2004/02/15/73366.aspx).)

ASP caches a lot more than that though.  It also pools script engines and caches compiled state.  Let me explain with an example. 

Suppose a request comes in for this page, time.asp:

 \<html\>  
\<% Response.write Now() %\>  
\</html\> 

Suppose further that this is the first-ever request for this page since the server was started.  

The thing above isn’t a legal script, so ASP parses out the non-script blocks and stores them in the Response object.  It then generates a real script that looks something like 

Response.WriteBlock 0  
Response.Write Now()  
Response.WriteBlock 1 

ASP also has a pool of created-but-not-initialized script engines sitting around.  I said in my last post that **when the script engines are not initialized they can be called on any thread,** so it does not matter that these engines were not created on whatever thread happened to be pulled out of the thread pool.  

ASP initializes the engine on the current thread and passes in the script code above.  The script engine compiles the script, runs it, and the ASP object model sends the resulting string off to the client. 

The script engine then gets uninitialized, but does not throw away its compiled state.  The compiled state – the proprietary bytecode form of the language – is maintained in memory because the second time someone asks for that page, ASP do not want to have to go through all the time and expense of translating the page to script and compiling the script to bytecode. 

The thread and the now-uninitialized engine go back to their respective pools. 

Now suppose a second request comes in for time.asp.  This time ASP pulls a thread out of the thread pool, notices that it has an engine with the compiled state just sitting there, attaches the engine to the thread, and runs the script again.  No time is spent creating the thread, creating the script engine, translating the page to code or compiling the script to bytecode. 

Suppose further that in the few milliseconds that ASP is running this script, a third request comes in for this page.  Then what happens?  There are two obvious possibilities: 

1\)      Wait for the current engine to finish, and start it right back up again when it is done.   
2\)      Fetch a new engine from the engine pool and start over – translate the page to script, compile, run. 

The trouble is, both of those are potentially slow.  The first in particular could be extremely slow. We needed a third option. 

In my previous post I discussed [the sorts of trouble you can get into with multi-threading](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx).  Note that the problem I described happens because two threads tried to both read *and* write a particular value.  If the value was read-only, then obviously there would not have been a problem.  You can read a read-only value from as many threads as you want\! 

Hey, once the script engine compiles a script into bytecode, that bytecode is read-only.  It’s never going to change again.  This means that two script engines on two different threads can share bytecode\! 

I lied earlier when I said that InterruptScriptThread was the only exception to our weird threading model.  We have another exception – at any time, you can call Clone from any thread.  This takes an existing script engine and gives you a new script engine with exactly the same compiled state, but able to run on a different thread. Each engine maintains its own runtime state – variable values, etc – but the compiled state is shared. 

So in this scenario, ASP **clones the running engine onto another thread and then runs the new engine concurrently**.  This is *somewhat* expensive – it has to create a new engine, after all – but is *much cheaper* than compiling up all that state again. 

Of course, ASP is much more complicated than that quick sketch.  What I’m getting at here is that there actually is some rhyme and reason to our bizarre-seeming choices of threading model.  

The script engine design was driven by the need to solve a very specific set of very disparate problems.  Without knowing what the problems were, the resulting design looks kind of random.  But once you know the history, well, I guess it looks a little *less* random.

UPDATEL: I've written a follow-up article on [how the ASP compilation model is affected by having multiple languages per page](http://blogs.msdn.com/ericlippert/archive/2004/02/19/76438.aspx).

