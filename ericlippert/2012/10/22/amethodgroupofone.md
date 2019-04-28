# A method group of one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/22/2012 6:24:00 AM

-----

I'm implementing the semantic analysis of dynamic expressions in Roslyn this week, so I'm fielding a lot of questions within the team on the design of the dynamic feature of C\# 4. A question I get fairly frequently in this space is as follows:

public class Alpha  
{  
  public int Foo(string x) { ... }  
}  
...  
dynamic d = whatever;  
Alpha alpha = MakeAlpha();  
var result = alpha.Foo(d);

How is this analyzed? More specifically, what's the type of local result?

If the receiver (that is, alpha) of the call were of type dynamic then there would be little we could do at compile time. We'd analyze the compile-time types of the arguments and emit a dynamic call site that caused the semantic analysis to be performed at runtime, using the runtime type of the dynamic expression. But that's not the case here. We know at compile time what the type of the receiver is. One of the design principles of the C\# dynamic feature is that if we have a type that is known at compile time, then at runtime the type analysis honours that. In other words, we only use the runtime type of the things that were actually dynamic; everything else we use the compile-time type. If MakeAlpha() returns a derived class of Alpha, and that derived class has more overloads of Foo, we don't care.

Because we know that we're going to be doing overload resolution on a method called Foo on an instance of type Alpha, we can do a "sanity check" at compile time to determine if we know that for sure, this is going to fail at runtime. So we do overload resolution, but instead of doing the full overload resolution algorithm (eliminate inapplicable candidates, determine the unique best applicable candidate, perform final validation of that candidate), we do a partial overload resolution algorithm. We get as far as eliminating the inapplicable candidates, and if that leaves one or more candidates then the call is bound dynamically. If it leaves zero candidates then we report an error at compile time, because we know that nothing is going to work at runtime.

Now, a seemingly reasonable question to ask at this point is: overload resolution in this case could determine that there is exactly one applicable candidate in the method group, and therefore we can determine statically that the type of result is int, so why do we instead say that the type of result is dynamic?

That appears to be a reasonable question, but think about it a bit more. If you and I and the compiler know that overload resolution is going to choose a particular method then *why are we making a dynamic call in the first place?* Why haven't we cast d to string? This situation is rare, unlikely, and has an easy workaround by inserting casts appropriately (either casting the call expression to int or the argument to string). Situations that are rare, unlikely and easily worked around are poor candidates for compiler optimizations. You asked for a dynamic call, so you're going to get a dynamic call.

That's reason enough to not do the proposed feature, but let's think about it a bit more deeply by exploring a variation on this scenario that I glossed over above. Eta Corporation produces:

public class Eta {}

and Zeta Corporation extends this code:

public class Zeta : Eta  
{  
  public int Foo(string x){ ... }  
}  
...  
dynamic d = whatever;  
Zeta zeta = new Zeta();  
var result = zeta.Foo(d);

Suppose we say that the type of result is int because the method group has only one member. Now suppose that in the next version, Eta Corporation supplies a new method:

public class Eta  
{  
  public string Foo(double x){...}  
}

Zeta corporation recompiles their code, and hey presto, suddenly result is of type dynamic\! Why should Eta Corporation's change **to the base class** cause the semantic analysis of code that uses a **derived** class to change? This seems unexpected. C\# has been carefully designed to avoid these sorts of "Brittle Base Class" failures; [see my other articles on that subject for examples of how we do that.](http://blogs.msdn.com/b/ericlippert/archive/tags/brittle+base+classes/)

We can make a bad situation even worse. Suppose Eta's change is instead:

public class Eta  
{  
  protected string Foo(double x){...}  
}

Now what happens? Should we say that the type of result is int when the code appears outside of class Zeta, because overload resolution produces a single applicable candidate, but dynamic when it appears inside, because overload resolution produces two such candidates? That would be quite bizarre indeed.

The proposal is simply too much cleverness in pursuit of too little value. We've been asked to perform a dynamic binding, and so we're going to perform a dynamic binding; the result should in turn be dynamic. The benefits of being able to statically deduce types of dynamic expressions does not pay for the costs, so we don't attempt to do so. **If you want static analysis then don't turn it off in the first place.**

**Next time:** the dynamic taint of method type inference.

