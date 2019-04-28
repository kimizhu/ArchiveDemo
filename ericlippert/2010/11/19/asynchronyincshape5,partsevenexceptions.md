# Asynchrony in C\# 5, Part Seven: Exceptions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/19/2010 8:47:53 AM

-----

Resuming where we left off (ha ha ha\!) after that brief interruption: exception handling in "resumable" methods like our coroutine-like asynchronous methods is more than a little bit weird. To get a sense of how weird it is, you might want to first refresh your memory of my [recent series on the design of iterator blocks](http://blogs.msdn.com/b/ericlippert/archive/tags/iterators/), particularly [the post about the difference between a "push" model and a "pull" model](http://blogs.msdn.com/b/ericlippert/archive/2009/07/23/iterator-blocks-part-five-push-vs-pull.aspx). Briefly though:

In a regular code block, a try block surrounding a normal "synchronous" call site observes any exceptions that occur within the call:

try { Q(); }  
catch { ... }  
finally { ... }

If Q() throws an exception then the catch block runs; when control leaves Q() by regular or exceptional means, the finally block runs. Nothing unusual here.

Now consider an iterator block that has been rewritten into a MoveNext method of an enumerator. When that thing is called it is called synchronously. If it throws an exception then the exception is handled by the nearest try-protected region on the call stack. But suppose the iterator block itself has **a try-protected region that yields control back to the caller:**

try { yield return whatever; }  
catch { ... }  
finally { ... }

The yield statement returns control back to the caller, but the return does *not* activate the finally block. And if an exception is thrown *in the caller* then the MoveNext() is no longer on the stack. Its exception handler has *vanished*. The exception model of iterator blocks is pretty weird. *The finally block only runs when control is in the MoveNext() method and leaves the try-protected region by some mechanism other than yield return*. Or when the enumerator is disposed early, which can happen if the caller throws an exception that activates a finally block that disposes the enumerator. In short: if the thing you've yielded control to has an exception that leaves the loop that is iterating the enumerator then **the *finally* block of the iterator probably runs, but the *catch* block does not**\! Bizarre. That's why we made it *illegal* for you to yield in a try block that has a catch.

So what on earth are we going to do for methods with "awaits" in them? The situation is like the situation with iterator blocks, but *even more bizarre* because of course **the asynchronous task can itself throw an exception**:

async Task M()  
{  
  try { await DoSomethingAsync(); }  
  catch { ... }  
  finally { ... }  
}

We *do* want it to be legal to await something in a try block that has a catch. Suppose DoSomethingAsync throws *before* it returns a task. No problem there; M is still on the stack, so the catch block runs. Suppose DoSomethingAsync returns a task. M signs up the rest of itself as the continuation of the task, and immediately returns *another* task to its caller. What happens when the job associated with the task returned by DoSomethingAsync is scheduled to run, and *it* throws an exception? *Logically* we want M to still be "on the stack" so that its catch and finally run, just like it would if DoSomething had been a synchronous call. (Unlike iterator blocks: we want the catch to run, not just the finally\!) But M is long gone; it has signed up a delegate that contains code that looks just like it as the continuation of a task, but M and its try block are vanished. *The task might not even be running on the thread that M ran on.* Heck, it might not even be running on the same *continent* if the task is actually farmed out to some service provider "in the cloud". What do we do?

I said a few episodes back that exception handling in continuation passing style is easy; you just pass around two continuations, one for the exceptional situation and one for the regular situation. That's not actually what we do here. Instead what we do is: if the job throws an otherwise-uncaught exception then *it is caught and the exception is stored in the task*. The task is then signaled as having completed unsuccessfully. When the continuation of the task resumes, we do a "goto" into the middle of the try block (somehow) and check to see if the task blew up. If it did, then we can *re-throw the exception right there*, and hey, this time there is a try-catch-finally that can handle the exception.

But suppose we do not handle the exception; maybe the catch block doesn't match. What do we do then? M's original caller is again, long gone; the continuation is probably being called by some top-level message pump somewhere. What do we do? Well, remember, **M returned a task**. We cache the exception *again* in *that* task, and then signal that task as having completed unsuccessfully. Thus the buck is passed to the caller, which is of course what exception throwing is all about: making your caller do the work of cleaning up your mess.

In short, M() is generated as something like this pseudo-C\#:

Task M()  
{  
  var builder = AsyncMethodBuilder.Create();  
  var state = State.Begin;  
  Action continuation = ()=\>  
  {  
    try  
    {  
      if (state == State.AfterDoSomething) goto AfterDoSomething;  
      try  
      {  
        var awaiter = DoSomethingAsync().GetAwaiter;  
        state= State.AfterDoSomething;;  
        if (awaiter.BeginAwait(continuation))  
          return without running the finally;  
      AfterDoSomething:  
        awaiter.EndAwait(); // throws an exception if the task completed unsuccessfully  
        builder.SetResult();  
        return;  
      }  
      catch { ... }  
      finally { ... }  
    }  
    catch (Exception exception)  
    {  
      builder.SetException(exception); // signal this task as having completed unsuccessfully  
      return;  
    }  
    builder.SetResult();  
  };  
  continuation();  
  return builder.Task;  
}

(Of course there are problems here; you cannot do a goto into the middle of a try block, the label is out of scope, and so on. Ve have vays of making the compiler generate IL that works; it doesn't have to be legal C\#. This is just a sketch.)

If the EndAwait throws an exception cached from the asynchronous operation then the catch and finally blocks run normally. If the inner catch block doesn't handle it, or throws another exception, then the outer catch block gets it, caches it in the task, and signals the task as having completed abnormally.

I have ignored several important cases in this brief sketch. For example, what if the method M is void returning? In that situation there is no task constructed for M, and so there is nothing to be signalled as completed unsuccessfully, and nowhere to cache the exception. What if DoSomethingAsync does a WhenAll on ten sub-tasks and two of them throw an exception? What about the same scenario but with WhenAny?

**Next time** I'll talk a bit about these cases, muse about exception handling philosophy in general, and ask you whether that philosophy gives good guidance or not. Then we'll take a short break for American Thanksgiving, and then pick up with some topic *other* than asynchrony.

