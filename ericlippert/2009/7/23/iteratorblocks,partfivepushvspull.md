# Iterator Blocks, Part Five: Push vs Pull

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/23/2009 10:05:00 AM

-----

A while back [I posted some commentary I did for the Scripting Summer Games where I noted that there is an isomorphism between "pull" sequences and "push" events.](http://blogs.msdn.com/ericlippert/archive/2009/06/26/iterators-at-the-summer-games.aspx) Normally you think of events as something that "calls you", pushing the event arguments at you. And normally you think of a sequence as something that you "pull from", asking it for the next value until you're done. But you can treat a stream of event firings as being a sequence of event argument objects. And similarly, you could implement a sequence iterator so that it called a method for every object in the collection. [Heck, you could implement all of LINQ on this model if you really wanted.](http://msmvps.com/blogs/jon_skeet/archive/2008/01/04/quot-push-quot-linq-revisited-next-attempt-at-an-explanation.aspx)

The implementation of iterator blocks is clearly on the "pull" paradigm. It didn't have to be. We could have gone with a sort of "inversion of control" approach. "Pull" iterators have to simulate coroutines with a little state machine that knows how to get back to the state where the code left off. But we already have a mechanism for code to "get back to the state where it left off" -- that's what you do every time you call a method\! You remember what you were doing, call the method, and then pick up where you left off. We could have done the same thing with iterators. We could say that this:

 

void Integers(int length, IObserver\<int\> observer)  
{  
  for (int i = 0; i \< length; ++i) yield return i;  
}

is a syntactic sugar for

 

void Integers(int length, IObserver\<int\> observer)  
{  
  for (int i = 0; i \< length; ++i) observer.Yield(i);  
  observer.Break();  
}

That is, we "raise an event" by calling the observer every time something yields, and raise another event when we're done.

This would be a much more trivial transformation to make than our current state machine approach, but since most people want sequence iteration on the "pull" model, we do the harder work to make that happen.

I said this had something to do with exception handling. What's the connection?

Did you notice something weird about how we handle finally blocks? Consider what happens with the "push model":

 

TextReader reader = File.Open(file);  
try  
{  
    blah blah blah yield the lines  
}   
finally  
{  
    reader.Close();  
}  

If something in the "blah blah blah" is realized as a call to observer.Yield(line), what happens if the code consuming the result throws? Easy. It is just a method call like any other. The call stack unwinds, we find the finally, the file gets closed, everything is good.

Now suppose this is realized as the MoveNext of a "pull" iterator. What happens when the code that is consuming the result throws? If we're consuming the result then we've *returned* from the call to MoveNext\! There is no "try", there is no "finally". And yet usually the finally gets executed anyways\! If the code that is consuming the results throws, odds are good that it was in a foreach; when foreach throws, the enumerator is disposed, and when the enumerator is disposed, we figure out which "finallys" were around when we last left MoveNext, and we run a special method that does whatever was in the pending finally blocks.

*This is pretty weird.* What's going on here is that for finally blocks, "pull" iterators have the same semantics as "push" iterators. When you yield from a try containing a finally, it looks like the finally is still "on the stack" and runs when the consumer throws.

So what if the try block has a catch?

The original designers of C\# 2.0 -- and remember, this is long before my time on the team -- had a huge debate over this. A debate which was repeated in miniature when I sent them all an email asking for the justification for this decision. There are three basic positions:

1\) Do not allow yield returns in try blocks at all. (Or blocks that are sugars for try blocks, like using blocks.) Use some other mechanism other than "finally" to express the idea of "this is the code that you run when the caller shuts down the enumeration."

2\) Allow yield returns in all try blocks.

3\) Allow yield returns in try blocks that have finally blocks, but not if they have catch blocks.

The down sides of (1) are that you have to come up with some syntax for representing the "finally" logic that isn't actually a "finally", and that it becomes more difficult to iterate over stuff that requires cleanup, like iterating over the contents of a file. This makes the feature both confusing and weak; we should avoid this option if at all possible.

The down side of (2) is that the exception handling behaviour beomes deeply, weirdly inconsistent between finally blocks and catch blocks. If you have a yield in a try block with a finally, and the consumer of the iteration throws, then the finally runs, as though you were on the "push" model. But if you have a yield in a try block with a catch, and the consumer of the iteration throws, then the catch does not run, because its not on the call stack. Users have a reasonable expectation that finally and catch work more or less the same way when an exception happens; that is, that control will be consistently transferred to a matching catch or the finally.

The down side of (3) is that the rule seems arbitrary and weird -- until you read five unnecessarily prolix blog entries that explain what on earth the design team was thinking.

Obviously, they picked (3), and now you know why.

Next time, we'll finish up this series with a look at unsafe code. Now that you know all this stuff, seeing why there's no unsafe code allowed in an iterator block is pretty straightforward by comparison.

