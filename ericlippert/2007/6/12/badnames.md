# Bad Names

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/12/2007 2:59:00 PM

-----

Over the past year and a half of working on the C\# compiler we’ve refactored a lot of crufty old code. But a surprising number of the “refactorings” have actually simply been renaming a structure, class, or method to something more sensible. The compiler is showing its age; there are a lot of structures, classes and methods inside the compiler which no longer have sensible names (if, in fact, they ever did). Going through the whole thing all at once and renaming everything without changing the semantics of any of the code is a fairly cheap and easy way to massively improve code readability. After all, what is easier to understand, FUNCBREC or StatementBindingContext? (I think FUNCBREC at one point stood for “function binding record”.)

One of the things we’ve been looking for is names that “smell bad”. Sometimes the thing simply needs to be renamed and that’s the end of the story. Sometimes, though, the existence of a bad name is a red flag in the code that points to some chunk of functionality which could use some serious thought and additional code refactoring applied to it.

Functions with names like DoTheOtherThingToIt are obviously badly named. There’s no way to look at the call site and have the faintest idea what’s happening on the callee side.  But there are more subtle signs of badness. I’ve been compiling a list of these red flag particles, and here is what we’ve come up with so far:

**Misc**:

This is indicative that the [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) has been violated.  Functions or structures which do “miscellaneous” work almost certainly do two or more things which are unrelated.

**Base/Core/Worker/Helper:**

These are often a sign of too much or too little abstraction. Suppose InferTypeSignature calls InferTypeSignatureWorker. If InferTypeSignatureWorker is doing all the work of inferring a type signature then move it back into InferTypeSignature. If it is not doing all the work, then describe what work it is doing, don’t just say that it’s working on something.

**Fake/Real:**

In the compiler we had a structure called FakeMethodSymbol, deriving from MethodSymbol. OK, it’s a symbol, it represents a method, got it. But in what sense is it a “fake” method? Is it a compiler-generated method as opposed to a user-generated method? Is it some kind of stub for a missing method? When is it legal to use as a MethodSymbol? Who knows? I sure didn’t. “Fake” doesn’t tell you anything about its purpose.

(It turned out that this is used when representing the method signature of an expanded method call that uses a C++ style “varargs” argument list. We’ve renamed it ExpandedVarargsMethodSymbol, and now it is extremely clear what that thing is for.)

**Special/Normal:**

Describe the property that makes the case special, rather than just calling it out as special. And try not to call things “normal”, just call them things. Rather than NormalFoo and SpecialFoo, have Foo and FrobbingFoo. Or have a NonFrobbingFoo and a FrobbingFoo both derived from abstract base class Foo.

**Ex/2:**

Oh, the pain. These should be used to solve COM interface versioning problems, and even then only as a last resort.

Those are the ones we've found in the compiler so far. What sort of bad-smelling name particles do you guys encounter in crufty code?

