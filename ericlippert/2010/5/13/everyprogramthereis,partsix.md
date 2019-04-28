# Every Program There Is, Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/13/2010 6:18:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx).\]

The problem we ran into with our grammar-enumeration technique is that the recursion does not clearly reduce the problem to a smaller problem. We need to find some way to reduce the problem to a smaller problem, while at the same time guaranteeing that we are in fact enumerating all possible programs defined by a given grammar.

I said before that there are two kinds of grammars: ambiguous and unambiguous. We massively prefer unambiguous grammars because they have the nice property that every possible program that matches the grammar has exactly one leftmost parse. Recall that we had the example of a grammar that produces sums:

 

S: N | S + N  
N: 1 | 2 | 3

and a leftmost derivation of 1 + 2 + 3:

S  
S + N  
S + N + N  
N + N + N  
1 + N + N  
1 + 2 + N  
1 + 2 + 3

This is the *only* way to produce the string 1 + 2 + 3 via leftmost substitutions in this grammar. Notice that there are seven lines in this substitution.

Hmm. This is the only substitution path which produces this string, and it has seven lines. Since the grammar is unambiguous, every *string in the grammar's language is associated with some specific integer which is the number of lines in its unique derivation.* And there have got to be a finite number of programs that require seven substitutions, since there are always a finite number of possible leftmost substitutions at every step in the derivation.

This is the "aha" insight: instead of looking at the length of the final string, we could look at *the length of the derivation required to produce the string*. That is, we can enumerate all the strings of terminals in a grammar by enumerating all the strings that take a single substitution, then all the strings that took two substitutions, then all the strings that took three substitutions… and so on.

Again, I want a concise notation for “all the strings of terminals whose derivations start with just the nonterminal X and have exactly n steps.” To mean that, I’m going to say X\[n\]. And again, to mean “the combination of all strings of form X which require n substitutions followed by all strings of form Y which require m substitutions”, I’ll say X\[n\]Y\[m\].  
  
Consider again our modification of the arithmetic grammar.

S: N | S P  
P: + N  
N: 1 | 2 | 3

How would we say “show me all the strings of the form S which result from *exactly* five substitutions”? That is, S\[5\]. We make one substitution to get either N or S P. That leaves four more substitutions we can perform on each of these two possible initial substitutions. So the solution is the set of strings formed by the union of N\[4\], S\[4\]P\[0\], S\[3\]P\[1\], S\[2\]P\[2\], S\[1\]P\[3\] and S\[0\]P\[4\].

Remember, we need three things to make a workable recursive solution: reduction of problem size, combination of subproblem solutions, and a trivial base case.

This time we really have reduced the problem size; each subproblem requires strictly fewer substitutions at every step of the way. Clearly we are combining solutions to subproblems. Do we have a trivial base case?

Yes. First, we know a priori that there are no strings in S\[0\] or P\[0\] because S and P are nonterminals; they always require at least one substitution to produce a string of terminals. We can solve this problem trivially: there are no such strings. And second, suppose we are trying to solve the problem P\[2\]. The solution to P\[2\] is the union of +\[1\]N\[0\] and +\[0\]N\[1\]. We know that N\[0\] is the empty set because that is “no substitutions on this nonterminal”. We also know that +\[1\] is empty because that is “one substitution of this terminal”, but there are no substitutions allowed on terminals. And we know that +\[0\] is the string +, because that is “no substitutions on this terminal”, which results in the terminal.

We forgot one thing, and that is to consider NIL, the special guy which produces neither terminals nor nonterminals. That’s easily taken care of; we simply define NIL\[0\] as being the set containing only the empty string and NIL\[n\] for any nonzero n as being the empty set; there are no strings which result from multiple substitutions on NIL. (Again, this essentially characterizes NIL as being a fancy way of writing the terminal which is the empty string; alternatively, we could have said that NIL is a nonterminal and defined NIL\[1\] as the set containing only the empty string and it then follows that NIL\[n\] is the empty set for all other n. It really doesn't matter which characterization we pick as long as we do so consistently.)

**Next time:** rewriting a grammar to make this easier.

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/10/every-program-there-is-part-five.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx).\]

