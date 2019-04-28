# As Timeless As Infinity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/15/2009 6:25:00 AM

-----

**User:** Recently I found out about a peculiar behaviour concerning division by zero in floating point numbers in C\#. It does not throw an exception, as with integer division, but rather returns an "infinity". Why is that?

**Eric:** As I've often said, "why" questions are difficult for me to answer. My first attempt at an answer to a "why" question is usually "because that's what the specification says to do"; this time is no different. The C\# specification says to do that in section 4.1.6. But we're only doing that because that's what the IEEE standard for floating point arithmetic says to do. We wish to be compliant with the established industry standard. See [IEEE standard 754-1985](http://en.wikipedia.org/wiki/IEEE_754-1985) for details. Most floating point arithmetic is done in hardware these days, and most hardware is compliant with this specification.

**User:** It seems to me that division by zero is a bug no matter how you look at it\!

**Eric:** Well, since clearly that is not how the members of the IEEE standardization committee looked at it in 1985, your statement that it must be a bug "no matter how you look at it" must be incorrect. Some industry experts do not look at it that way.

**User:** Good point. What motivated this design decision?

**Eric:** I wasn't there; I was busy playing Jumpman on my Commodore 64 at the time. But my educated guess is that **it is desirable for all possible operations on all floats to produce a well-defined float result**. Mathematicians would call this a "closure" property; that is, the set of floating point numbers is "closed" over all operations.

Positive infinity seems like a reasonable choice for dividing a positive number by zero. It seems plausible because of course the limit of 1 / x as x goes to zero (from above) is "positive infinity", so why shouldn't 1/0 be the number "positive infinity"?

Now, speaking *as a mathematician*, I find that argument specious. A thing and its limit need not have any particular property in common; it is fallacious to reason that just because, say, a sequence has a particular limit that a fact about the limit is also a fact about the sequence. Mathematically, "positive infinity" (in the sense of a limit of a real-valued function; let's leave transfinite ordinals, hyperbolic geometry, and all of that other stuff out of this discussion) is not a number at all and should not be treated as one; rather, it's a terse way of saying "the limit does not exist because the sequence diverges upwards".

When we divide by zero, essentially what we are saying is "solve the equation x \* 0 = 1"; the solution to that equation is not "positive infinity", it is "I cannot because there is no solution to that equation". It's just the same as asking to solve the equation "x + 1 = x" -- saying "x is positive infinity" is not a solution; there is no solution.

But speaking *as a practical engineer* who uses floating point numbers to do an imprecise approximation of ideal arithmetic, this seems like a perfectly reasonable choice.

**User:** But surely it is impossible for the hardware to represent "infinity".

**Eric:** It certainly is possible. You've got 32 bits in a single-precision float; that's over four billion possible floats. All bit patterns of the form

?11111111???????????????????????

are reserved for "not-a-number" values. That's over sixteen million possible NaN combinations. Two of those sixteen million NaN bit patterns are reserved to mean positive and negative infinity. Positive infinity is the bit pattern 01111111100000000000000000000000 and negative infinity is 11111111100000000000000000000000.

**User:** Do all languages and applications use this convention of division-by-zero-becomes-infinity?

**Eric:** No. For example, C\# and JScript do but VBScript does not. VBScript gives an error if you do that.

**User:** Then how do language implementors get the desired behaviour for each language if these semantics are implemented by the hardware?

**Eric:** There are two basic techniques. First, many chips which implement this standard allow the programmer to make float division by zero an exception rather than an infinity. On the 80x87 chip, for example, you can use bit two of the precision control register to determine whether division by zero returns an infinity or throws a hardware exception.

Second, if you don't want it to be a hardware exception but do want it to be a software exception, then you can check bit two of the status register after each division; it records whether there was a recent divide-by-zero event.

The latter strategy is used by VBScript; after we perform a division operation we check to see whether the status register recorded a divide-by-zero operation; if it did, then the VBScript runtime creates a divide-by-zero error and the usual VBScript error management process takes over, same as any other error.

Similar bits exist for other operations that seem like they might be better treated as exceptions, like numeric overflow.

The existence of the "hardware exception" bits creates problems for the modern language implementor, because we are now often in a world where code written in multiple languages from multiple vendors is running in the same process. Control bits on hardware are the ultimate "global state", and we all know how irksome it is to have global, public state that random code can stomp on.

For example: I might be misremembering some details, but I seem to recall that Delphi-authored controls set the "overflows cause exceptions" bit. That is, the Delphi implementors did not use the VBScript strategy of "try it, allow it to succeed, and check to see whether the overflow bit was set in the status register". Rather, they used the "make the hardware throw an exception and then catch the exception" strategy. This is deeply unfortunate. When a VBScript script calls a Delphi-authored control, the control flips the bit to force exceptions but it never "unflips" it. If, later on in the script, the VBScript program does an overflow, then we get an unhandled hardware exception because the bit is still set, even though the Delphi control might be long gone\! I fixed that by saving away the state of the control register before calling into a component and restoring it when control returns. That's not ideal, but there's not much else we can do.

**User:** Very enlightening\! I will be sure to pass this information along to my coworkers. I would be delighted to see a blog post on this.

**Eric:** And here you go\!

