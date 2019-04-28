# Why is deriving a public class from an internal class illegal?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/13/2012 9:32:26 AM

-----

In C\# it is illegal to declare a class D whose base class B is in any way less accessible than D. I'm occasionally asked why that is. There are a number of reasons; today I'll start with a very specific scenario and then talk about a general philosophy.

Suppose you and your coworker Alice are developing the code for assembly Foo, which you intend to be fully trusted by its users. Alice writes:

public class B  
{  
  public void Dangerous() {...}  
}

And you write

public class D : B  
{  
... other stuff ...  
}

Later, Alice gets a security review from Bob, who points out that method Dangerous could be used as a component of an attack by partially-trusted code, and who further points out that customer scenarios do not actually require B to be used directly by customers in the first place; B is actually only being used as an implementation detail of other classes. So in keeping with the [principle of least privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege), Alice changes B to:

internal class B  
{  
  public void Dangerous() {...}  
}

Alice need not change the accessibility of Dangerous, because of course "public" means "public to the people who can see the class in the first place".

So now what should happen when Alice recompiles before she checks in this change? The C\# compiler does not know if you, the author of class D, intended method Dangerous to be accessible by a user of public class D. On the one hand, it is a public method of a base class, and so it seems like it should be accessible. On the other hand, the fact that B is internal is evidence that Dangerous is supposed to be inaccessible outside the assembly. A basic design principle of C\# is that **when the intention is unclear, the compiler brings this fact to your attention by failing**. The compiler is identifying yet another form of the Brittle Base Class Failure, which long-time readers know has [shown up in numerous places in the design of C\#](http://blogs.msdn.com/b/ericlippert/archive/tags/brittle+base+classes/).

Rather than simply making this change and hoping for the best, you and Alice need to sit down and talk about whether B really is a sensible base class of D; it seems plausible that either (1) D ought to be internal also, or (2) D ought to [favour composition over inheritance](http://en.wikipedia.org/wiki/Composition_over_inheritance). Which brings us to my more general point:

**More generally**: the inheritance **mechanism** is, [as we've discussed before](http://blogs.msdn.com/b/ericlippert/archive/2011/09/19/inheritance-and-representation.aspx), simply the fact that all heritable members of the base type are also members of the derived type. But the inheritance relationship **semantics** are intended to model the "is a kind of" relationship. It seems reasonable that if D is a kind of B, and D is accessible at a location, then B ought to be accessible at that location as well. It seems strange that you could only use the fact that "a Giraffe is a kind of Animal" at specific locations.

In short, this rule of the language encourages you to use inheritance relationships to **model the business domain semantics** rather than as a **mechanism for code reuse**.

Finally, I note that as an alternative, **it *is* legal for a public class to implement an internal interface**. In that scenario there is no danger of accidentally exposing dangerous functionality from the interface to the implementing type because of course the interface is not associated with any functionality in the first place; an interface is logically "abstract". Implementing an internal interface can be used as a mechanism that allows public components in the same assembly to communicate with each other over "back channels" that are not exposed to the public.

