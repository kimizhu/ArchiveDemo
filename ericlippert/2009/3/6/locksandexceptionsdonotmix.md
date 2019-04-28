# Locks and exceptions do not mix

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/6/2009 11:41:00 AM

-----

A couple years ago I wrote a bit about how our codegen for the lock statement could sometimes lead to situations in which an unoptimized build had [different potential deadlocks](http://blogs.msdn.com/ericlippert/archive/2007/08/17/subtleties-of-c-il-codegen.aspx) than an optimized build of the same source code. This is unfortunate, so we've fixed that for C\# 4.0. However, all is still not [rainbows, unicorns and Obama](http://iamchriscollins.com/badpaintingsofbarackobama/images/4.jpg), as we'll see.

Recall that lock(obj){body} was a syntactic sugar for

 

var temp = obj;  
Monitor.Enter(temp);  
try { body }  
finally { Monitor.Exit(temp); }

The problem here is that if the compiler generates a no-op instruction between the monitor enter and the try-protected region then it is possible for the runtime to throw a thread abort exception after the monitor enter but before the try. In that scenario, the finally never runs so the lock leaks, probably eventually deadlocking the program. It would be nice if this were impossible in unoptimized and optimized builds.

In C\# 4.0 we've changed lock so that it now generates code as if it were

 

bool lockWasTaken = false;  
var temp = obj;  
try { Monitor.Enter(temp, ref lockWasTaken); { body } }  
finally { if (lockWasTaken) Monitor.Exit(temp); }

The problem now becomes someone else's problem; the implementation of Monitor.Enter takes on responsibility for atomically setting the flag in a manner that is immune to thread abort exceptions messing it up.

So everything is good now, right?

Sadly, no. It's *consistently bad* now, which is better than being inconsistently bad. But there's an enormous problem with this approach. By choosing these semantics for "lock" we have made a potentially unwarranted and dangerous choice on your behalf; **implicit in this codegen is the belief that a deadlocked program is the worst thing that can happen**. That's not necessarily true\! **Sometimes deadlocking the program is the better thing to do -- the lesser of two evils. **

The *purpose* of the lock statement is to help you protect the integrity of a mutable resource that is shared by multiple threads. But suppose an exception is thrown halfway through a mutation of the locked resource. Our implementation of lock does not magically roll back the mutation to its pristine state, and it does not complete the mutation. Rather, control *immediately* branches to the finally, releasing the lock and allowing every other thread that is patiently waiting to *immediately* view the messed-up partially mutated state\! If that state has privacy, security, or human life and safety implications, the result could be very bad indeed. In that case it is *possibly* better to deadlock the program and protect the messed-up resource by denying access to it entirely. But that's obviously not good either.

Basically, we all have a difficult choice to make whenever doing multi-threaded programming. We can (1) automatically release locks upon exceptions, exposing inconsistent state and living with the resulting bugs (bad) (2) maintain locks upon exceptions, deadlocking the program (arguably often worse) or (3) carefully implement the bodies of locks that do mutations so that in the event of an exception, the mutated resource is rolled back to a pristine state before the lock is released. (Good, but hard.)

This is yet another reason why **the body of a lock should do as little as possible**. Usually the rationale for having small lock bodies is to get in and get out quickly, so that anyone waiting on the lock does not have to wait long. But an even better reason is because small, simple lock bodies minimize the chance that the thing in there is going to throw an exception. It's also easier to rewrite mutating lock bodies to have rollback behaviour if they don't do very much to begin with.

And of course, this is yet another reason why aborting a thread is pure evil. Try to never do so\!

