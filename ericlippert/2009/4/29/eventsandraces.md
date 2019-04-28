# Events and Races

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/29/2009 8:00:00 AM

-----

Here’s a question [similar](http://stackoverflow.com/questions/786383/c-events-and-thread-safety) to one I saw on [stackoverflow](http://www.stackoverflow.com/) the other day. Suppose you have an event:

 

public event Action Foo;

The standard pattern for firing this event is:  

Action temp = Foo;  
if (temp \!= null)  
      temp();

What the heck is up with that? Why not just call “Foo()” ?

First off, this pattern ensures that the thing that is invoked is not the null delegate reference, which would cause a null reference exception to be thrown. But if that’s what we want, then surely we could have skipped the temporary variable – just “if (Foo \!= null) Foo();” would do. Why the temp?

The temporary variable ensures that this is “thread safe”. If the event’s handlers are being modified on another thread it is possible for Foo to be non-null at the point of the check, then set to null on a different thread, and then the invocation throws.

Using the temporary variable effectively makes a copy of the current set of event handlers. Remember, **multi-cast delegates are immutable**; when you add or remove a handler, you *replace* the existing multi-cast delegate object with a *different* delegate object that has different behaviour. You do not modify an existing *object*, you modify the *variable* that stores the event handler. Therefore, stashing away the current reference stored in that variable into a temporary variable effectively makes a copy of the current state.

Is that clear? Make sure it is, because now things start to get really confusing.

A common criticism of this pattern is that it trades one race condition for another. Let’s consider that scenario again more carefully:

The event handler contains a single handler, H. Thread Alpha makes a copy of the delegate to H in temp and determines that it is not null. Thread Beta decides that H must no longer be called when the event is fired, so it sets the handler to null. Thread Beta then assumes that H will never be called, and destroys a bunch of state that H needs to execute successfully. Thread Alpha then regains control and executes H, which behaves crazily since its necessary state has been destroyed. Thread Beta’s attempt to ensure that H is not called has been defeated by the race condition.

A frequently-stated principle of good software design is that code which calls attention to bugs by throwing exceptions is better than code which hides bugs by muddling on through and doing the wrong thing. This code has a race condition that results in incorrect behaviour. Perhaps an application of that principle is to stop using a temporary and instead crash if it races. That is to say, this code has a failure mode; surely it is better to highlight that failure with a crisp exception than to turn it into crazy wrong behaviour.

That seems plausibly argued, I agree, but the conclusion is wrong.

Suppose we remove the temporary variable but keep the null check. Does that *solve* the problem? No\! We still have the same race conditions. Suppose the event handler delegate contains a reference to H. Thread Alpha checks to see whether it is null; it is not. Thread Alpha pushes the object to invoke on the runtime stack. *Between the push of the delegate value and the call to invoke it, thread Beta sets the event handler to null.* And once again, H will be invoked after thread Beta has removed the handler.

Suppose we remove the temporary *and* the null check and just invoke the delegate directly. Does that help? No, we *still* have the same race condition. The contents of the event handling variable can still be changed between the push of the delegate object onto the stack and the invocation of the delegate.

**Removing the code around the call site does not decrease the number of race conditions in the code, and it does increase the number of crashes. Worse, doing so makes the race condition harder to detect by shrinking the window in which the race can occur without eliminating it.**

Removing the null check and/or the temporary is a bad idea; the already-bad situation is only made worse.

Essentially there are two problems here that are being conflated. The two problems are:

1\) The event handler delegate can be null at any time.  
2\) A “stale” handler can be invoked even after it has been removed.

These two problems are actually orthogonal and have different solutions. The onus for solving the first problem is laid upon the *code which does the invocation*; it is required to ensure that it does not dereference null. The store-in-temporary-and-test pattern ensures that null is never dereferenced. (There are other ways to solve this problem; for example, initializing the handler to have an empty action that is never removed. But doing a null check is the standard pattern.)

The onus for solving the second problem is laid upon *the code being invoked*; **event handlers are required to be robust in the face of being called even after the event has been unsubscribed.** In the scenario I described, **the bug is actually in H**. It needs to be robust enough to check to see whether the state it needs is still there, and bail out cleanly if it is not. (Or, alternatively, some additional locking mechanism needs to be implemented which ensures that the code which fires the event cooperates with the code that changes the event handlers to ensure the desired behaviour.)

The point though is that the "null ref problem" and the "stale handler problem" are two separate problems that are easily confused because the symptoms of both arise at the exact same call site code. You've got to solve *both* problems if you want to do threadsafe events. How you do so is up to you; just don't confuse the solution of one problem for a solution of the other.

(Thanks to Microsofties Levi Broderick, Chris Burrows, Curt Hagenlocher and Wolf Logan; this article is based on a conversation about their analysis of this pattern.)

