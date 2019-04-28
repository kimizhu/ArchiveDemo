# What’s the difference between a destructor and a finalizer?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/21/2010 6:20:00 AM

-----

Today, another dialogue, and another episode of my ongoing series "what's the difference?"

**What’s the difference, if any, between a “destructor” and a “finalizer”?**

Both are mechanisms for cleaning up a resource when it is no longer in use. When I was asked this, at first I didn’t think there was a difference. But some Wikipedia searches turned up a difference; the term “destructor” is typically used to mean a **deterministically-invoked** cleanup, whereas a “finalizer” runs when the **garbage collector** says to run it.

**Doesn’t that mean that the C\# spec uses the term “destructor” incorrectly?**

Yes, by these definitions, the C\# spec gets it wrong. What we call a “destructor” in the spec is actually a finalizer, and what we call the “Dispose()” method invoked by a “using” statement is in fact a “destructor”.

The CLI spec calls the finalizer by its right name.

**Why did the authors of the C\# spec get it wrong?**

I don't know, but I can guess. I have two guesses. Guess \#1 is that on May 12th, 1999 there was not a Wikipedia article clearly describing the subtle difference between these two concepts. That's because there wasn't a Wikipedia. Remember back when there wasn't a Wikipedia? Dark ages, man. The error might simply have been an honest mistake, believing that the two terms were identical. Heck, for all I know, the two terms were identical on May 12th, 1999, and the difference in definitions only evolved later, as it became obvious that there was a need to disambiguate between eager/deterministic and lazy/nondeterministic cleanup methods. Anyone who has more historical perspective on this than I do, feel free to chime in here. Guess \#2 is that on May 12th, 1999, the language design committee wished to leave open the possibility that a C\# "destructor" could be implemented as something other than a CLR finalizer. That is, the "destructor" was designed to be a C\# language concept that did not necessarily map one-to-one with the CLR’s "finalizer" concept. When designing a language at the same time as the framework it sits atop is also being designed, sometimes you want to insulate yourself against late-breaking design changes in your subsystems. Deliberately preventing name conflation is one way to do that. **What’s your sudden obsession with May 12th, 1999 about?**

The language committee's notes for May 12th 1999 read in part:

> We're going to use the term "destructor" for the member which executes when an instance is reclaimed. Classes can have destructors; structs can't. Unlike in C++, a destructor cannot be called explicitly. Destruction is non-deterministic – you can't reliably know when the destructor will execute, except to say that it executes at some point after all references to the object have been released. The destructors in an inheritance chain are called in order, from most descendant to least descendant.  There is no need (and no way) for the derived class to explicitly call the base destructor. The C\# compiler compiles destructors to the appropriate CLR representation.  For this version that probably means an instance finalizer that is distinguished in metadata. 

Notice that this supports my hypothesis that the language design team was attempting to insulate themselves from becoming tied to a particular CLR term.

