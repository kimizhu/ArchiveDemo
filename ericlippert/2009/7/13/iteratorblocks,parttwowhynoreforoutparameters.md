# Iterator Blocks, Part Two: Why no ref or out parameters?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/13/2009 9:35:00 AM

-----

A long and detailed discussion of how exactly we implement iterator blocks would take me quite a while, and would duplicate work that has been done well by others already. I encourage you to start with Raymond’s series, which is a pretty gentle introduction: [part 1](http://blogs.msdn.com/oldnewthing/archive/2008/08/12/8849519.aspx), [part 2](http://blogs.msdn.com/oldnewthing/archive/2008/08/13/8854601.aspx), [part 3](http://blogs.msdn.com/oldnewthing/archive/2008/08/14/8862242.aspx). If you want a more detailed look at how this particular sausage is made, [Jon’s article is quite in-depth](http://csharpindepth.com/Articles/Chapter6/IteratorBlockImplementation.aspx).

To make a long story short, we implement iterators by:

  - Spitting a class that implements the relevant interfaces.
  - Hoisting locals of the iterator block to become fields of the class.
  - Rewriting the iterator block as the “MoveNext” method of the class.
  - Tracking the point where the last yield happened in the MoveNext method, and branching to that point directly using a switched “goto” when MoveNext is called.
  - Moving the logic in “finally” blocks (or equivalents, like the auto-generated finally blocks associated with lock statements and using statements) into the Dispose method, to ensure that stuff gets cleaned up when the iteration is completed or abandoned.

This is not the only implementation strategy we could have pursued. As I said last time, we could have implemented full-blown continuations and built the whole thing out of them; any conceivable control flow can be built out of continuations. But getting continuations right requires a lot of support from the framework, and would be very expensive. We can get by with less, so we do.

Similarly, we could have built this out of “coroutine [fibers](http://en.wikipedia.org/wiki/Fiber_\(computer_science\))”. A fiber is like a thread in that it has its own stack, but the code on the fiber itself decides when the fiber is done running. Again, this is a strategy – implement coroutines – that I mentioned last time which we rejected as too costly and difficult to get right in the CLR. Again, we can get by with less, so we do.

But this choice of a feasible, cost-effective implementation strategy then drives restrictions back into the design space; we have to lose some generality in order to make this strategy work.

The first thing that we lose is the ability to have iterator blocks be in methods which take ref or out parameters. In order to preserve the values of local variables and formal parameters across calls to MoveNext, we hoist the locals and parameters into fields of a compiler-generated class.

[As I discussed earlier, the CLR designers had an implementation goal of being able to take advantage of the performance characteristics of the system stack.](http://blogs.msdn.com/ericlippert/archive/2009/05/04/the-stack-is-an-implementation-detail-part-two.aspx) What happens when you store a reference in a field? The reference could be to something on the stack, but the field could live longer than the thing on the stack. The designers of the CLR were faced with four choices:

  - Build a system which is as fragile and horrid as unmanaged C code, a language in which accidentally storing a reference to something with too-short lifetime is a frequent source of awful crashing bugs and data corruptions.
  - Disallow (\*) taking references to locals; any local taken as a reference must be hoisted to become a field on a garbage-collected reference type. Safe, but slow.
  - Disallow storing references in fields.
  - Invent something else that solves this problem.  

The CLR designers chose the third option. Which means that our choice of hoisting parameters to fields immediately runs into a problem; we have no safe way of storing references to other variables in a field. The reference could be to something on the stack, but the iterator could live longer than the variable that is on the stack. A confluence of implementation choices has now driven an unfortunate and seemingly arbitrary restriction into the design. We lose a bit of generality in order to gain simplicity of implementation and higher performance in common cases.

Now think about that decision from the perspective of the larger goal of the feature. Remember, the feature is not “make any possible method into an iterator block”. The feature is “make it easier to write code that enumerates the members of a collection”. An “out” or “ref” parameter exists solely to enable a method to mutate an outside variable and therefore pass back information to the caller when the method terminates.

Why would you want to do that in a method whose sole purpose is to produce the sequence of values stored in a collection? What would the meaning of mutations to the variable even be if the iterator block mutates the variable multiple times as the iterator is running? Normally that kind of thing is not visible (modulo goofy multithreading scenarios) until the method is done, but an iterator block returns control to its caller many times before its done.

This seems like a scenario which is uncommon, strange and hard to implement. And furthermore, there are ways to achieve mutation of outer variables during iteration without passing in a ref to a variable; you could pass in an Action delegate. Instead of this illegal program:

 

IEnumerable\<int\> Frob(out string s)  
{  
  yield return 1;  
  s = "goodbye";  
}  
void Grob()  
{  
  string s = "hello";  
  foreach(int i in Frob(out s)) …  
}

you can always do the equivalent thing with this legal program:

 

IEnumerable\<int\> Frob(Action\<string\> setter)  
{  
  yield return 1;  
  setter("goodbye");  
}  
void Grob()  
{  
  string s = "hello";  
  foreach(int i in Frob(x=\>{s=x;})) …  
}

Since making the more general feature work given our implementation constraints is (1) hard or impossible, (2) clearly a corner case, not a mainline scenario, and (3) has a possible workaround, it’s a no-brainer to put a restriction on the design and disallow ref and out parameters.

Next time, why no yield in a finally block?

\*\*\*\*\*\*\*\*\*\*\*

(\*) Or, weaker, make it allowed but unverifiable. That is pretty much the same thing from the compiler implementers perspective; we try to never generate unverifiable code unless it is specifically marked “unsafe”.

