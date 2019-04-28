# Calling constructors in arbitrary places

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/28/2010 7:10:00 AM

-----

C\# lets you call another constructor from a given constructor, but only before the body of the calling constructor runs:  

public C(int x) : this(x, null)  
{  
  // …  
}  
public C(int x, string y)  
{  
  // …  
}

Why can you call another constructor at the beginning of a constructor block, but not at the end of the block, or in the middle of the block? Well, let's break it down into two cases. (1) You're calling a "base" constructor, and (2) you're calling a "this" constructor. For the "base" scenario, it's quite straightforward. You almost never want to call a base constructor after a derived constructor. That's an inversion of the normal dependency rules. Derived code should be able to depend on the base constructor having set up the "base" state of the object; the base constructor should never depend on the derived constructor having set up the derived state.

Suppose you're in the second case. The typical usage pattern for this scenario is to have a bunch of constructors that take different arguments and then all "feed" into one master constructor (often private) that does all the real work. Typically the public constructors have no bodies of their own, so there's no difference between calling the other constructor "before" or "after" the empty block. Suppose you're in the second case *and* you are doing work in each constructor, *and* you want to call other constructors at some point other than the start of the current constructor. In that scenario you can easily accomplish this by extracting the work done by the different constructors into methods, and then *calling the methods* in the constructors in whatever order you like. That is superior to inventing a syntax that allows you to call other constructors at arbitrary locations. There are a number of design principles that support this decision. Two are: 1) Having two ways to do the same thing creates confusion; it adds mental cost. We often have two ways of doing the same thing in C\#, but in those situations we want the situation to "pay for itself" by having the two different ways of doing the thing each be compelling, interesting and powerful features that have clear pros and cons. (For example, "query comprehensions" vs "fluent queries" are two very different-looking ways of building a query.)  Having a way to call a constructor the way you'd call any other method seems like having two ways of doing something -- calling an initialization method -- but without a compelling or interesting "payoff".  2) We'd have to add new language syntax to do it. New syntax comes at a very high cost; it's got to be designed, implemented, tested, documented -- those are our costs. But it comes at a higher cost to you because *you have to learn what the syntax means, otherwise you cannot read or maintain other people's code*.  That's another cost; again, we only take the huge expense of adding syntax if we feel that there is a clear, compelling, large benefit for our customers.  I don't see a huge benefit here.

In short, achieving the desired construction control flow is easy to do without adding the feature, and there's no compelling benefit to adding the feature. No new interesting representational power is added to the language.

