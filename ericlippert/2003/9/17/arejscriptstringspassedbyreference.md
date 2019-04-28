# Are JScript strings passed by reference?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2003 5:24:00 PM

-----

Yesterday I asked "are JScript strings passed **by reference** (like JScript objects) or **by value** (like JScript numbers)?"

 

 

Trick question\!  It doesn't matter, because you can't change a string.  I mean, suppose they were passed by reference -- how would you know?  You can't have two variables refer to the "same" string and then change that string.  Strings in JScript are like numbers -- immutable primitive values.

 

 

Of course, "under the covers" we actually have to pass the strings somehow.  Generally speaking, strings are passed by reference where possible, as it is much cheaper in both time and memory to pass a pointer to a string than to make a copy, pass the value, and then destroy the copy.

