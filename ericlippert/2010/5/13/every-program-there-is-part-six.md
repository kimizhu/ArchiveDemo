<div id="page">

# Every Program There Is, Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/13/2010 6:18:00 AM

-----

<div id="content">

<div class="mine">

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx).\]

The problem we ran into with our grammar-enumeration technique is that the recursion does not clearly reduce the problem to a smaller problem. We need to find some way to reduce the problem to a smaller problem, while at the same time guaranteeing that we are in fact enumerating all possible programs defined by a given grammar.

I said before that there are two kinds of grammars: ambiguous and unambiguous. We massively prefer unambiguous grammars because they have the nice property that every possible program that matches the grammar has exactly one leftmost parse. Recall that we had the example of a grammar that produces sums:

<span class="code"> </span>

S: N | S + N  
N: 1 | 2 | 3

and a leftmost derivation of <span class="code">1 + 2 + 3</span>:

<span class="code">S  
S + N  
S + N + N  
N + N + N  
1 + N + N  
1 + 2 + N  
1 + 2 + 3</span>

This is the *only* way to produce the string <span class="code">1 + 2 + 3 </span>via leftmost substitutions in this grammar. Notice that there are seven lines in this substitution.

Hmm. This is the only substitution path which produces this string, and it has seven lines. Since the grammar is unambiguous, every *string in the grammar's language is associated with some specific integer which is the number of lines in its unique derivation.* And there have got to be a finite number of programs that require seven substitutions, since there are always a finite number of possible leftmost substitutions at every step in the derivation.

This is the "aha" insight: instead of looking at the length of the final string, we could look at *the length of the derivation required to produce the string*. That is, we can enumerate all the strings of terminals in a grammar by enumerating all the strings that take a single substitution, then all the strings that took two substitutions, then all the strings that took three substitutions… and so on.

Again, I want a concise notation for “all the strings of terminals whose derivations start with just the nonterminal <span class="code">X</span> and have exactly n steps.” To mean that, I’m going to say <span class="code">X\[n\]</span>. And again, to mean “the combination of all strings of form <span class="code">X</span> which require n substitutions followed by all strings of form <span class="code">Y</span> which require m substitutions”, I’ll say <span class="code">X\[n\]Y\[m\]</span>.  
  
Consider again our modification of the arithmetic grammar.

<span class="code">S: N | S P  
P: + N  
N: 1 | 2 | 3</span>

How would we say “show me all the strings of the form <span class="code">S</span> which result from *exactly* five substitutions”? That is, <span class="code">S\[5\]</span>. We make one substitution to get either <span class="code">N</span> or <span class="code">S P</span>. That leaves four more substitutions we can perform on each of these two possible initial substitutions. So the solution is the set of strings formed by the union of <span class="code">N\[4\]</span>, <span class="code">S\[4\]P\[0\]</span>, <span class="code">S\[3\]P\[1\]</span>, <span class="code">S\[2\]P\[2\], <span class="code">S\[1\]P\[3\]</span></span> and <span class="code">S\[0\]P\[4\]</span>.

Remember, we need three things to make a workable recursive solution: reduction of problem size, combination of subproblem solutions, and a trivial base case.

This time we really have reduced the problem size; each subproblem requires strictly fewer substitutions at every step of the way. Clearly we are combining solutions to subproblems. Do we have a trivial base case?

Yes. First, we know a priori that there are no strings in <span class="code">S\[0\]</span> or <span class="code">P\[0\]</span> because <span class="code">S</span> and <span class="code">P</span> are nonterminals; they always require at least one substitution to produce a string of terminals. We can solve this problem trivially: there are no such strings. And second, suppose we are trying to solve the problem <span class="code">P\[2\]</span>. The solution to <span class="code">P\[2\]</span> is the union of <span class="code">+\[1\]N\[0\]</span> and <span class="code">+\[0\]N\[1\]</span>. We know that <span class="code">N\[0\]</span> is the empty set because that is “no substitutions on this nonterminal”. We also know that <span class="code">+\[1\]</span> is empty because that is “one substitution of this terminal”, but there are no substitutions allowed on terminals. And we know that <span class="code">+\[0\]</span> is the string <span class="code">+</span>, because that is “no substitutions on this terminal”, which results in the terminal.

We forgot one thing, and that is to consider <span class="code">NIL</span>, the special guy which produces neither terminals nor nonterminals. That’s easily taken care of; we simply define <span class="code">NIL\[0\]</span> as being the set containing only the empty string and <span class="code">NIL\[n\]</span> for any nonzero n as being the empty set; there are no strings which result from multiple substitutions on <span class="code">NIL</span>. (Again, this essentially characterizes NIL as being a fancy way of writing the terminal which is the empty string; alternatively, we could have said that NIL is a nonterminal and defined NIL\[1\] as the set containing only the empty string and it then follows that NIL\[n\] is the empty set for all other n. It really doesn't matter which characterization we pick as long as we do so consistently.)

**Next time:** rewriting a grammar to make this easier.

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx).\]

</div>

</div>

</div>

