# The Root of All Evil, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/29/2006 5:45:00 PM

-----

There were a number of interesting comments to yesterday's posting about premature optimization.

Several readers pointed out that there are various options we could take:

1.  Do nothing. Maintain backwards compatibility and spec incompatibility.
2.  Change the spec to exactly match the current implementation.
3.  Change the spec and the implementation.
4.  Make the weird behaviour a warning now and an error later.
5.  Fix the implementation to match the specification, but add another compiler switch to go back to the previous behaviour.

 

These all have various pros and cons, and even for a relatively trivial spec incompatibility such as this one, we think very hard about what the right thing to do is.

Option \#2 sounded good, and I actually was an advocate of it for a few hours yesterday, right up until I dug into the optimization and discovered exactly what it was doing. If we were to change the specification to exactly match the behaviour, then the specification would read like this:

Implicit Enumeration Conversions

There is an implicit conversion from any "magic zero" to any enumerated type. "Magic zero" is recursively defined as follows:

  - the integer literal zero, or
  - any magic zero + - | ^ any compile-time integer constant expression equal to zero, or
  - any compile-time integer expression \* & any magic zero, or
  - any magic zero / % any non-zero compile-time constant integer expression, or
  - any magic zero \* & any non-zero compile-time constant integer expression

 

This means for example, that (8\*(0+(7-7))) is a magic zero, but (8\*((7-7)+0) is not.

I think we can all agree that we don't want this nonsense in the specification. Changing the spec to exactly match the buggy behaviour leads to a crazy spec.

We could change both the specification and the implementation. We could, for instance, say that any compile-time constant integer zero is implicitly convertible to any enumerated type. This means that nonsense like (8\*(0+(7-7))) is assignable to enum, but at least it also means that (8\*((7-7)+0) is too.

Adding a new warning is a tempting idea, and we might end up doing it. Doing so means that it is easier to turn it into an error in a future release. New warnings, however, are expensive. They add new code paths to test, we have to test all the scenarios to make sure the warning always appears when it should, warnings have to be translated into a dozen languages, they have to be documented, the documentation has to be translated... it's not cheap.

Adding a compiler switch is also a possibility, but also expensive. The more switches there are, again, more code paths, more tests, bigger testing matrix, the switch has to be documented, and so on.

So now you see why premature optimization is the root of all evil. I've spent the better part of two days wrestling with this thing. Why? Because suppose one of these guys is in a lambda expression that needs to be translated into an expression tree. I want to make for darn sure that the translation is correct, and the more "optimizations" that happen before that transformation, the more the transformation is likely to be not as the user intended\! But I can't remove the optimizations without breaking the type system\!

UPDATE: The thrilling conclusion is that we have made all constant-according-to-the-spec zero integers convertible to any enum. This is a spec violation, but at least it is a consistent spec violation. In doing so I also introduced a bug, which I believe did ship to customers but which we have since fixed, whereby some "non-integer zeros" (like default instances of structs, or null references) were convertible to any enum. I apologize for the error.

I also challenged you folks to come up with a situation where this premature optimization can screw up definite assignment checking. Reader Carlos found two.

int aa; int bb = 12; int cc;  
int dd = (aa \* 0) + 12;  
if (bb \* 0 == 0) cc = 123;  
Console.WriteLine(cc);  

According to the spec, aa should be flagged as "used before definitely assigned" (even though really it doesn't matter), but because the premature optimization throws away the aa, the flow checker never sees it. Also according to the spec, cc should be flagged as "used before definitely assigned" (even though it always is assigned), but because the optimization throws away the bb, the flow checker does not realize that the conditional is not a compile-time-constant.

The important thing here is not that the code gets it wrong. Indeed, one could make the argument that it gets it right. The problem is that it gets it right for the wrong reasons – it gets it right by accident, in violation of the specification. The definite assignment specification isn't supposed to be perfect, it's supposed to be practical and understandable and predictable and implementable. Undocumented extensions to it make it hard to know whether any two implementations of the spec will agree.

