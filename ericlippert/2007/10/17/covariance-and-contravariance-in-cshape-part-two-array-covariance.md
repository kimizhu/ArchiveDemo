<div id="page">

# Covariance and Contravariance in C\#, Part Two: Array Covariance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/17/2007 11:47:00 AM

-----

<div id="content">

<div class="mine">

C\# implements variance in two ways. Today, the broken way.

Ever since C\# 1.0, **arrays where the element type is a reference type are covariant**. This is perfectly legal:

<span class="code"> </span>

Animal\[\] animals = new Giraffe\[10\];

Since <span class="code">Giraffe</span> is smaller than <span class="code">Animal</span>, and “make an array of” is a covariant operation on types, <span class="code">Giraffe\[\]</span> is smaller than <span class="code">Animal\[\]</span>, so an instance fits into that variable.

Unfortunately, **this particular kind of covariance is broken**. It was added to the CLR because Java requires it and the CLR designers wanted to be able to support Java-like languages. We then up and added it to C\# because it was in the CLR. This decision was quite controversial at the time and I am not very happy about it, but there’s nothing we can do about it now.

Why is this broken? Because *it should always be legal to put a <span class="code">Turtle</span> into an array of <span class="code">Animal</span>s*. With array covariance in the language and runtime you cannot *guarantee* that an array of <span class="code">Animal</span>s can accept a <span class="code">Turtle</span> *because the backing store might actually be an array of <span class="code">Giraffe</span>s*.

This means that we have turned a bug which could be caught by the compiler into one that can only be caught at runtime. This also means that every time you put an object into an array we have to do a run-time check to ensure that the type works out and throw an exception if it doesn’t. That’s potentially expensive if you’re putting a zillion of these things into the array.

Yuck.

Unfortunately, we’re stuck with this now. <span class="code">Giraffe\[\]</span> is smaller than <span class="code">Animal\[\]</span>, and that’s just the way it goes.

I would like to take this opportunity to clarify some points brought up in comments to [Part One](http://blogs.msdn.com/ericlippert/archive/2007/10/16/covariance-and-contravariance-in-c-part-one.aspx).

First, by "subtype" and "supertype" I mean "is on the chain of base classes" for classes and "is on the tree of base interfaces" for interfaces. I do not mean the more general notion of "is substitutable for". And by “bigger than” and “smaller than” I explicitly do NOT mean “is a supertype of” and “is a subtype of”. It is the case that every subclass is smaller than its superclass, yes, but *not vice versa*. That is, it is not the case that every smaller type is a subtype of its larger type. <span class="code">Giraffe\[\]</span> is smaller than both <span class="code">Animal\[\]</span> and <span class="code">System.Array</span>. Clearly <span class="code">Giraffe\[\]</span> is a subtype of <span class="code">System.Array</span>, but it is **emphatically** **not** a *subtype* of <span class="code">Animal\[\]</span>. Therefore the “is smaller than” relationship I am defining is more general than the “is a kind of” relationship. I want to draw a distinction between assignment compatibility (smaller than) and inheritance (subtype of).

Next time we’ll discuss a kind of variance that we added to C\# 2.0 which is *not* broken.

</div>

</div>

</div>

