# An "is" operator puzzle, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/23/2012 10:12:00 AM

-----

It is possible for a program with some local variable x:

bool b = x is FooBar;

to assign true to b at runtime, even though there is no conversion, implicit or explicit, from x to FooBar allowed by the compiler\! That is,

FooBar foobar = (FooBar)x;

would not be allowed by the compiler in that same program.

Can you create a program to demonstrate this fact? This is not a particularly hard puzzle but it does illustrate some of the subtleties of the "is" operator that we'll discuss in the next episode.

