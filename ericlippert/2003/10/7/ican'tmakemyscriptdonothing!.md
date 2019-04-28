# I can't make my script do nothing\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/7/2003 12:55:00 PM

-----

Yes, the title is grammatical. A few days ago I was discussing the semantics of **data that isn't there**. Today I want to talk a little about **programs that do nothing**. What do you do when you want a program to pause briefly, for whatever reason?

 

Those of you who like me cut your programming teeth on Commodore PET Basic know the answer. A for loop will do the trick.

 

for(x = 0 ; x \< 1000000 ; ++x) {}

 

Of course, that has the problem that the time spent paused is shorter on faster machines.  The obvious improvement waits until a given number of milliseconds have passed:

 

for(var startTime = new Date(); new Date() - startTime \> 1000 ; ) {}

 

The problem with both these techniques is that this is **busy waiting**.  The program may *appear* to be paused but it is actually executing as furiously as ever.  The processor is probably pegged to 100% but doing entirely pointless work.  In a multithreaded, multiprocess operating system busy waiting is downright rude -- the processor could be tending to other tasks like redrawing the screen or background processing.

 

Really what you want to do is to ask the operating system to put the process asleep for a bit and then wake it up later, right?  That way the program pauses but the thread scheduler is free to run something more important. And indeed, there's a Win32 API that does just that, Sleep.

 

But there is no way to call win32 APIs directly from script, so the scriptable "go to sleep" method needs to either be built in to the language, or (the horror\!) in an ActiveX object. So why the heck do VBScript and JScript not have a "go to sleep" method built in?  

 

Well, actually, there were good reasons why we didn't. The principal reason is because I lied above.  You do NOT want to put the process to sleep and wake it up again later, because while the script is "sleeping", **you still might want event handlers to run.**

 

The way events work in Apartment Threaded COM objects such as initialized script engines (see my earlier entries for a refresher on how the script engine threading model works) is pretty simple. When the user clicks on a button and raises a button event, what actually happens is a bunch of messages are deposited in the thread's message queue.  When the message queue is pumped, the messages tell the COM plumbing to call the event sinks listening to the event sources.

 

So what happens when a thread is asleep?  No message loop runs on the thread\!  How could it?  *It's asleep*.  So you put your script to sleep for ten seconds, a user presses a button, and then waits at least ten seconds for the message loop to get pumped?  That seems kind of suboptimal.  The sleep method needs to pump a message loop occasionally so that events get handled.

 

 

It gets worse.  What if one of those event sinks causes a script error, which is then reported to the host, and the host decides to shut down the script engine? Ten seconds later, the script wakes up and keeps running\! Oh, the pain.

 

To properly implement a "go to sleep" method you need to know all kinds of details about the desired message processing, event handling, error handling and multi-threading semantics of the host application.  We were very worried that we'd add a method to the script engines which, when used in IE, or ASP, or WSH, or some third-party script host, completely screw up the carefully implemented host.

 

 

Hence, no "go to sleep" method in the languages.  If the host implementor thinks that a "go to sleep" method is necessary, they can implement one and add it to their object model, as we did for WSH.

