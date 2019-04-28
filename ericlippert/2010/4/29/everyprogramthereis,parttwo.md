# Every Program There Is, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/29/2010 7:08:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/26/every-program-there-is-part-one.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/03/every-program-there-is-part-three.aspx).\]

Suppose we want to come up with a CFG for numbers with additions. Consider this very simple grammar with only one nonterminal. We’ll say that an expression is either a number, or a sum of two expressions:

 

X: 1 | 2 | 3 | X + X

We can make the following series of substitutions, each time substituting the **leftmost** remaining nonterminal:

 

X  
X + X  
X + X + X  
1 + X + X  
1 + 2 + X  
1 + 2 + 3

We can also make this series of substitutions, again, every time substituting for the **leftmost** remaining nonterminal:

 

X  
X + X  
1 + X            \<-- different\!  
1 + X + X  
1 + 2 + X  
1 + 2 + 3

How interesting; we had two rather similar but nonetheless different “leftmost” derivations for the same sequence of terminals\! CFGs for which there exist strings of terminals that have two different derivations are called “ambiguous” grammars. It is often extremely difficult to correctly parse an ambiguous CFG, though of course we are not attempting to parse here. But it is also difficult to use an ambiguous CFG to generate all the strings in a language because what sometimes happens is you end up generating the same string more than once. Ideally for our purposes we would like to guarantee that a given string is only generated once. If the CFG is non-ambiguous then we know that trying all possible combinations of rule applications will not lead to the same string generated twice.

Unfortunately, the general problem of ambiguity turns out to be provably unsolvable. There is no *general* way to (1) look at a grammar and determine whether it is ambiguous, or (2) take an ambiguous CFG and find an “equivalent” unambiguous CFG. However, not all is lost. There are ways to prove that particular grammars are unambiguous, even though there is no general solution. And there are lots of common sneaky tricks for making the sorts of grammars we typically encounter in programming languages into unambiguous grammars.  In this particular case we could say:

 

S: N | S + N  
N: 1 | 2 | 3

And now the only leftmost derivation for 1 + 2 + 3 is

 

S  
S + N  
S + N + N  
N + N + N  
1 + N + N  
1 + 2 + N  
1 + 2 + 3

I’m not going to prove that this is an unambiguous grammar; that would take us too far afield. Just take my word for it, this is unambiguous. From now on we’ll be trying to produce only unambiguous grammars.

**Next time:** saying nothing requires extra work

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/26/every-program-there-is-part-one.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/03/every-program-there-is-part-three.aspx).\]

