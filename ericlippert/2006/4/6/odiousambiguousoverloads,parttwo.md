# Odious ambiguous overloads, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/6/2006 1:13:00 PM

-----

There were a number of ideas in the comments for what [we should do about the unfortunate situation I described yesterday](http://blogs.msdn.com/ericlippert/archive/2006/04/05/569085.aspx). We considered all of these options and more.  I thought I might talk a bit about the pros and cons of each.

I suggested **do nothing**.  We shipped with this behaviour in C\# 2.0, the world didn't come to an end, we can keep on shipping with it. The pro side is that this is the cheapest and easiest thing to do.  The con side is that as a reader pointed out, this is a gross "gotcha" just waiting for someone to stumble upon it.

DrPizza suggested that we **come up with a syntax** that disambiguates between the two ambiguous cases.  On the pro side, the problem would be clearly solved.  On the con side, this might be a breaking change (because suddenly one of the methods that used to bind one way might start binding the other way, depending on what the choice of syntax was.)

Also on the con side, adding new language syntax is something that we do not do lightly.  There is considerable work involved in changing the specification, and this is kind of an edge case.

Ayende Rahien suggested that we **add an attribute** that hints to the compiler which is the right binding. This has the nice property of not introducing new language syntax, but is kind of ugly.

These are both good ideas but unfortunately, as it turns out, neither of these work for technical reasons which I will go into in detail below.

Chris D suggested that **the constructed interface should be treated as an interface with only one method**. That's a really nice idea, but unfortunately it doesn't work with the CLR definition of generics.  An interface with n methods has n vtable slots that need filling, period, and n is invariant under construction. We'd have to change both the semantics of constructed interfaces and their implementation in the CLR, which means changing both the C\# spec and the CLR spec, and that's way too much work for this edge case.

Kyle Lahnakoski suggested that it **simply be an error**.  There are several ways this could be an error.

It could be an error to have an unconstructed generic type which could *possibly* be constructed to produce a duplicated signature.  This was how generics originally worked in C\#, but we decided that we could allow these cases and we can't really go back on that decision now – that would be a huge breaking change. There are many serializable generic objects which use a pattern of having a method that takes a type parameter to overload a method that takes a serialization object parameter, and all of those would suddenly be errors.

It could be an error to construct a generic type so as to *actually* produce a duplicate signature, but again, we decided to allow that in C\# 2.0 and it would be a large breaking change to go back now.

It could be an error to have any explicit implementation of an interface method which matches ambiguously.  This would be a smaller breaking change, but still breaking.

What do any of these breaking changes buy the user though?  Not more functionality, but less.  Basically we'd be saying "you could do this before, and now you can't, too bad." That's a hard sell.

Carlos suggested unification – that is, **have an explicit implementation map to every method that it matches, not just the first one**.  This is a breaking change, but it seems like the most natural thing to do.  Since an implicit implementation maps to every interface slot that it matches, why shouldn't an explicit implementation do the same? This means adding new language to the spec, but no new syntax.

This is what we decided to do, and I tried to implement it, only to discover that, uh, actually that stuff I told you yesterday about the implicit implementation matching the first one in source code order is a bit of a mistruth. That's what I thought when I first looked at the problem, but upon closer examination, something deeper was wrong.

In fact what is going on here is that when the CLR loads the class for the first time, it does a verification check to ensure that the class does in fact implement every method of every interface that it says it does.  It is the CLR, not C\#, which matches the explicit implementation to the first matching slot in the constructed interface type\! It just so happens that since metadata for the interface is emitted in source-code order, that the CLR appears to do the match in source code order.  It actually does it in a CLR-implementation-defined order that just happens to be source code order.

To understand why we can't fix this, you need to understand how explicit implementations are represented in metadata.  In the CLR metadata for a class there is a table called the MethodImpl table which says "method blah of class foo explicitly implements method bar of interface IFred".  The "method bar of interface IFred" portion of that mapping may be either a token to a method of the unconstructed interface (a "MethodDef") , or it may be a token representing a method on the constructed interface (a "MemberRef").

We can't use a MethodDef because there is a signature mismatch between the class's implementation and the method on the unconstructed interface.  One takes an int, the other takes a type parameter, and the CLR flags that as illegal. So that's out.

But we can't use the latter because the MemberRef identifies the constructed interface method by – you guessed it – its signature\!  And now we're in the same hideous bind all over again.  We have no way to tell the CLR "*no, really, use the generic-substituted version of the method"*, because **it can't tell the two signatures apart either**.

To fix this problem by either unifying, or adding a syntax/attribute to disambiguate, we need some way to represent that unification/disambiguation in metadata. But the CLR simply lacks the concept of "method token for constructed method identified by something more unique than signature". 

Though we can certainly suggest that this is a weakness in the CLR metadata design, waiting around for that to be addressed to fix this edge-case problem seems like a non-starter. Given that we can't fix it in the current CLR the way we'd like, and making it an error seems like an egregious overreaction that buys the user nothing, but at the same time doing nothing is leaving a gotcha in the code waiting to happen, I took the only course of action left:

 

Warning: Explicit interface implementation 'I1\<int\>.M(int)' matches more than one interface member. Which interface member is actually chosen is implementation-dependent. Consider using a non-explicit implementation instead." 

Unfortunately, sometimes that's the best you can do.

