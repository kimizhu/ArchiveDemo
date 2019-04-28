# Careful with that axe, part one: Should I specify a timeout?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/22/2010 6:41:00 AM

-----

[![Careful](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/ShouldIspecifyatimeout_CF1B/Careful_3.jpg "Careful")](http://indieexpress.com/AFIDigitalInterviewCAREFULWITHTHATAXE.htm)

(This is part one of a two-part series on the dangers of aborting a thread. [Part two is here](http://blogs.msdn.com/b/ericlippert/archive/2010/02/25/careful-with-that-axe-part-two-what-about-exceptions.aspx).)

The other day, six years ago, I was was talking a bit about [how to decide whether to keep waiting for a bus](http://blogs.msdn.com/ericlippert/archive/2003/09/24/i-m-a-traveling-man-don-t-tie-me-down.aspx), or to give up and walk. It led to a quite interesting discussion on [the old JoS forum](http://discuss.fogcreek.com/techInterview/default.asp?cmd=show&ixPost=1547&ixReplies=9). But what if the choice isn’t “wait for a bit then give up”, instead it is “wait for a bit, and then take an axe to the thread”? A pattern I occasionally see is something like I’ve got a worker thread that I started up, I ask it to shut down, and then I wait for it to do so. If it doesn’t shut down soon, take an axe to it:

 

this.running = false;  
if (\!workerThread.Join(timeout))  
    workerThread.Abort();

Is this a good idea?

It depends on just how badly the worker thread behaves and what it is doing when it is misbehaving.

If you can guarantee that the work is short in duration, for whatever 'short' means to you, then you don't need a timeout. If you cannot guarantee that, then I would suggest first rewriting the code so that you *can* guarantee that; life becomes much easier if you know that the code will terminate quickly when you ask it to.

If you cannot, then what's the right thing to do? The assumption of this scenario is that **the worker is ill-behaved and does not terminate in a timely manner when asked to**. So now we've got to ask ourselves "is the scenario that the worker is *slow by design*, *buggy*, or *hostile*?"

In the first option, the worker is simply doing something that takes a long time and for whatever reason, cannot be interrupted. What's the right thing to do here? I have no idea. This is a terrible situation to be in. Presumably the worker is not shutting down quickly because doing so is dangerous or impossible. In that case, what are you going to do when the timeout times out? You've got something that is dangerous or impossible to shut down, and its not shutting down in a timely manner. Your choices seem to be

(1) do nothing  
(2) wait longer  
(3) do something impossible. Preferably before breakfast.  
(4) do something dangerous  
  
Choice one is identical to not waiting at all; if that’s what you’re going to do then why wait in the first place? Choice two just changes the timeout to a different value; this is question begging. By assumption we're not waiting forever. Choice three is impossible. That leaves “do something dangerous”. Which seems… dangerous.

Knowing what the right thing to do in order to minimize harm to user data depends upon the exact circumstances that are causing the danger; analyze it carefully, understand all the scenarios, and figure out the right thing to do. There’s no slam-dunk easy solution here; it will depend entirely on the real code running.

Now suppose the worker is supposed to be able to shut down quickly, but does not because it has a bug. Obviously, if you can, fix the bug. If you cannot fix the bug -- perhaps it is in code you do not own -- then again, you are in a terrible fix. You have to understand what the consequences are of not waiting for already-buggy-and-therefore-unpredictable code to finish before disposing of the resources that you know it is using right now on another thread. And you have to know what the consequences are of terminating a thread while a buggy worker thread is still busy doing heaven only knows what to operating system state.

If the code is *hostile* and is *actively resisting being shut down* then you have already lost. You cannot halt the thread by normal means, and you cannot even reliably thread abort it. There is no guarantee whatsoever that aborting a hostile thread actually terminates it; the owner of the hostile code that you have foolishly started running in your process could be doing all of its work in a finally block or other constrained region which prevents thread abort exceptions.

The best thing to do is to never get into this situation in the first place; if you have code that you think is hostile, either do not run it at all, or run it in its own process, and terminate *the process*, not *the thread* when things go badly.

In short, there's no good answer to the question "what do I do if it takes too long?" You are in a *terrible* situation if that happens and there is no easy answer. Best to work hard to ensure you don't get into it in the first place; only run **cooperative, benign, safe code** that always shuts itself down cleanly and rapidly when asked. **[Careful with that axe, Eugene](https://www.youtube.com/watch?v=tMpGdG27K9o).**

Next time, what about exceptions?

(This is part one of a two-part series on the dangers of aborting a thread. [Part two is here](http://blogs.msdn.com/b/ericlippert/archive/2010/02/25/careful-with-that-axe-part-two-what-about-exceptions.aspx).)

