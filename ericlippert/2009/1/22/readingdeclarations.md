# Reading Declarations

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/22/2009 12:28:27 PM

-----

Wow, lots of good "hmm" moments in the comments to yesterday's post. Keep them coming\!

Many of these resonated strongly with me. One in particular was thinking back to the day when I finally internalized the reality that in C/C++, the declaration "int \* px;" does not break down "int \*" and "px", but rather "int" and "\*px".

Perhaps you all find this perfectly straightforward, but it takes me a little longer to figure all this stuff out. (And hence my motto: "Eric Lippert figures it out... eventually".) The conceptual leap that I had to make was this:

When you say "int x;" that means "any time you see x, it refers to a storage location that can contain an int". That's straightforward. When you see "int \* px;" the tempting thing to do is to reason "any time you see px, it refers to a storage location that can contain an int\*", but that is the wrong way to look at it. The insight is to shift your focus so that the conceptualization follows the same pattern as the "int x;" case. That is "any time you see **\*px**, it refers to a storage location that can contain an int".

Everything then follows from that insight. What can go into px? Anything that you can dereference and get an integer variable out of as a result\!

Once you start taking the declaration apart correctly, other confusions drop away. Like, I was always confused by const:

"const int y = 123;" means that y refers to a read-only storage location for an int. What does "const int \* py = whatever;" mean?  Does it mean that py is a pointer to an integer and **the pointer** does not change, or that **the integer pointed** to does not change? When you take apart the declaration correctly it becomes clear. It means "\*py refers to an read-only storage location for an int".

The same logic works for arrays. "int rx\[10\];" means "rx\[something\] refers to integer storage". "const int \*p\[10\];" means "\*p\[something\]" refers to const integer storage", and so on.

This also explains why "int \*px, x" gives you a pointer to an integer and an integer, not two pointers to integers. This says "\*px and x both refer to integer variables".

Now, perhaps I am not the only person to find this rather confusing, which motivates the completely different way we do it in C\#. In C\# the type of the variable is on the left hand side of the declaration and the name of the variable is on the right hand side, period. In C\# you say "int\*\[\] px;" and that means "px is a variable of type int\*\[\]".  Types can always be decomposed unambiguously in C\#; this is an array of pointers to int.

Of course, it helps that in C\# you cannot take a pointer to an array, and array types (being reference types) cannot be made nullable, so the number of possible combinations of type-modifying suffixes is very limited.

