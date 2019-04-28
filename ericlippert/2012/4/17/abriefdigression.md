# A brief digression

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/17/2012 7:04:00 AM

-----

Before we continue our exploration of truthiness in C\#, a brief digression. I mentioned last time the "knights and knaves" puzzles of logician Raymond Smullyan. Though I do enjoy those puzzles, my favourite of his puzzles are his chess puzzles, and my second favourite are his combinatory logic puzzles. Here's an example of the latter, adapted from the puzzle on page 134 of Satan, Cantor and Infinity.

We have an alphabet consisting of the letters S, E, R, M, K and V. You can make strings out of these letters; a "string" is defined as a ordered sequence of letters drawn from the given alphabet; the sequence can be of any finite length. Some strings are said to "name" other strings.

I'll use "x" and "y" as variables that stand in for strings, and furthermore we can assume that x and y never stand for empty strings. (And that moreover, in the fourth rule, y is a string with at least two characters.) The rules of naming are:

  - SxE names x
  - if x names y then Rx names yy
  - if x names y then Mx names SyE
  - if x names y then Kx names the string formed by removing the leftmost character from y
  - if x names y then Vx names the string formed by reversing the letters of y

So, for example, SRME names RM, by the first rule. Because SRME names RM, we know that RSRME names RMRM by the second rule. And thus KRSRME names MRM.

There has been a small amount of confusion in the comments; note that I am not saying that some strings are "legal" and some are "illegal" in any way; rather, I am saying that some strings just happen to "name" other strings. A string that does not name any other string is still a perfectly "good" string; no string is more or less valid than any other. "RM" for example does not name any string.

The challenge -- which probably will not be too hard for computer programmers, but will be a bit tricky for non-programmers who haven't read the fifteen pages of easier puzzles leading up to this one -- is to **find a string that names itself**. Smullyan's solution is 18 letters long and he challenges the reader to find a shorter one. I have found a 12 letter solution. I conjecture that 12 letters is the smallest such string, but I have not proven it. I'll put both Smullyan's solution and my solution in the comments. Good luck\!

UPDATE: There is a **nine**-letter solution, and it is minimal. Nice work\!

