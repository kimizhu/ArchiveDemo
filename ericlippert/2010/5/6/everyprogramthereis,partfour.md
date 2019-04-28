# Every Program There Is, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/6/2010 7:13:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/03/every-program-there-is-part-three.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx).\]

Remember, the point of this exercise was to come up with a device to help me test parts of my compiler. To test binary trees as we saw we can generate all trees of size one, then all trees of size two, then all trees of size three, and so on. This technique has some nice properties. First off, we know that we are exploring the space of binary trees without skipping any; for any particular tree, we will eventually get to it. Second, we are guaranteed to get the small ones first; it seems likely that if an algorithm works for all trees of size five or less, then it works for all trees period. And third, at no point does the algorithm run off into infinity; there are no infinite loops to worry about. There is a combinatorial explosion to worry about, since there are lots of possible trees, but each one can be generated in finite time. It would be nice to have similar properties for code which produces every sentence given some grammar.

The obvious brute-force way to do this involves building a parser for the grammar. That is, generate all length-one strings, parse them, and discard the ones that don’t parse legally. Then generate all length-two strings, and so on. The problem with this approach is that in order to generate, say, class c : b { }, even if you restrict yourself to an alphabet containing only lowercase letters, spaces, and punctuation, you have to generate and parse about a trillion trillion strings, the vast majority of which will be obvious non-programs like aaaaaaaaaaaa},c. This method is far too slow.

A smarter way to go would be to generate all strings that consist of one terminal, parse them, discard the illegal ones, then generate all strings that consist of two terminals, and so on. That is, start by parsing a, class, {, and so on. That’s way better; it only takes about a million steps to get to class c : b { }. But that’s still pretty slow, and we have to write a parser. What would be nicer is to simply not have to ever generate illegal programs in the first place.

What if instead we had some way to say “generate me all legal programs with one terminal; ok, now generate me all programs with two terminals”, and so on? That is, don’t create a candidate and test it, just create all the legal candidates. That would work. But how to do it?

Let’s go back to our first problem: generating strings for all the binary trees. A grammar for that language is:

 

T: x | ( T T )

We can use the same technique we did before\! Remember, before we generated all the binary trees of size four by saying that they were:

 

{all binary trees of size 0 } followed by {all binary trees of size 3}  
{all binary trees of size 1 } followed by {all binary trees of size 2}

and so on. Well, all binary trees strings with, say, eight terminals are:

 

( {all strings with 0 terminals} {all strings with 6 terminals} )  
( {all strings with 1 terminal} {all strings with 5 terminals} )

and so on. That is, we take out the two terminals for the surrounding parens, and then find all the combinations of substitutions that eventually lead to exactly six more on the inside. Clearly we have reduced a problem of size k to multiple problems of size less than k-1, and so this is amenable to a recursive solution. Right?

**Next time**: Am I right, or have I made a logical error somewhere above?

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/03/every-program-there-is-part-three.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx).\]

