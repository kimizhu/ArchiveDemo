<div id="page">

# Every Program There Is, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/10/2010 7:16:00 AM

-----

<div id="content">

<div class="mine">

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/06/every-program-there-is-part-four.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/13/every-program-there-is-part-six.aspx).\]

**Last time:** Clearly we have reduced a problem of size k to multiple problems of size less than k-1, and so this is amenable to a recursive solution. Right?

Wrong. (The “clearly” was a dead giveaway.) Remember, three things are necessary for a recursive solution. First, there has to be a way of reducing a problem of some size to a related problem of **smaller size**. Second, there has to be a way of **combining solutions** to smaller problems into a solution to a larger problem. And third, the smallest possible problem has to admit a non-recursive solution; **the recursion has to stop somewhere**.

Have we got all of that? We seem to, but let’s try a different example. We previously discussed a grammar for arithmetical sums. Let’s make a minor modification to that:

<span class="code"> </span>

S: N | S P  
P: + N  
N: 1 | 2 | 3

I hope you agree that this is clearly the same language, just with a slightly different grammar. Now how would we go about answering the question “what are all the strings in this grammar of form S with three terminals? I want a concise way of saying that, so let’s say that <span class="code">S\<3\></span> means that. To say “the set of strings which is the combination of all possible strings of <span class="code">X\<n\></span> followed by all possible strings of <span class="code">Y\<m\></span>” I’ll say  <span class="code">X\<n\>Y<span class="code">\<m\></span></span>

So, what is <span class="code">S\<3\></span> ? It is the combination of <span class="code">N\<3\></span>, <span class="code">S\<0\>P\<3\></span>, <span class="code">S\<1\>P\<2\></span>, <span class="code">S\<2\>P\<1\> </span>and <span class="code">S\<3\>P\<0\></span>.

Uh oh. We have not met the first condition; we “reduced” the problem of “what are all the strings of size three that match S” to a set of problems that requires us to solve that problem\! In fact, the subproblem <span class="code">S\<3\>P\<0\></span> is solvable if we know that we can skip it *because there are no such strings of the form P with zero terminals*. But how do we know that? In this particular case it is obvious that substitution of P never produces <span class="code">NIL</span>, but in general we have no way of knowing that.

Heck, we can’t even satisfy the third condition; the problem of working out <span class="code">S\<0\></span> is “reduced” to the problem of solving both <span class="code">N\<0\></span> and <span class="code">S\<0\>P\<0\></span>, which is a *larger* problem\!

We might be able to make some progress if we can annotate the grammar to say “by the way, I happen to know that this production rule always produces at least one terminal, so you don’t need to consider the zero cases.” That is doable, but it is somewhat vexing to have to do so.

Is there another way?

**Next time:** Why yes, yes there is.

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/06/every-program-there-is-part-four.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/13/every-program-there-is-part-six.aspx).\]

</div>

</div>

</div>

