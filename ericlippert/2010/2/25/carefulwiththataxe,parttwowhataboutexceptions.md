# Careful with that axe, part two: What about exceptions?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/25/2010 6:59:00 AM

-----

(This is part two of a two-part series on  the dangers of aborting a thread. [Part one is here](http://blogs.msdn.com/b/ericlippert/archive/2010/02/22/should-i-specify-a-timeout.aspx).)

Suppose you’re shutting down the worker thread we were talking about last time, and it throws an exception? What happens?

Badness, that’s what. What to do about it?

As in our previous discussion, it is better to not be in this situation in the first place: write the worker code so that it does not throw. If you cannot do that, then you have two choices: handle the exception, or don't handle the exception.

Suppose you don't handle the exception. As of I think CLR v2, an unhandled exception in a worker thread shuts down the whole application. The reason being, in the past what would happen is you'd start up a bunch of worker threads, they'd all throw exceptions, and you'd end up with a running application with no worker threads left, doing no work, and not telling the user about it. It is better to force the author of the code to handle the situation where a worker thread goes down due to an exception; doing it the old way effectively hides bugs and makes it easy to write fragile applications.

Suppose you do handle the exception. Now what? Something on another thread threw an exception, which is by definition an unexpected, exceptionally bad error condition. You now have no clue whatsoever that any of your data is consistent or any of your program invariants are maintained in any of your subsystems. So what are you going to do? There's hardly anything safe you can do at this point.

The question is "what is best for the user in this unfortunate situation?" It depends on what the application is doing. It is entirely possible that the best thing to do at this point is to simply aggressively shut down and tell the user that something unexpected failed. That might be better than trying to muddle on and possibly making the situation worse, by, say, accidentally destroying user data while trying to clean up.

Or, it is entirely possible that the best thing to do is to make a good faith effort to preserve the user's data, tidy up as much state as possible, and terminate as normally as possible.

Both today’s question and the one from last time are specific versions of the more general question "what do I do when my subsystems running on worker threads do not behave themselves?" If your subsystems are unreliable, either *make them reliable*, or *have a policy for how you deal with an unreliable subsystem, and implement that policy*. That's a vague answer I know, but that's because dealing with an unreliable subsystem is an inherently awful situation to be in. How you deal with it depends on the nature of its unreliability, and the consequences of that unreliability to the user's valuable data. There are no easy one-size-fits-all answers here, unfortunately.

(This is part two of a two-part series on  the dangers of aborting a thread. [Part one is here](http://blogs.msdn.com/b/ericlippert/archive/2010/02/22/should-i-specify-a-timeout.aspx).)

