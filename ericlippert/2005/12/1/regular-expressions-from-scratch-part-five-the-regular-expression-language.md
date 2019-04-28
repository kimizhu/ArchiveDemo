<div id="page">

# Regular Expressions From Scratch, Part Five: The Regular Expression Language

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/1/2005 10:00:00 AM

-----

<div id="content">

<div class="mine">

Now things start to get really weird.

**Definition 9**: Take any alphabet S. The **regular expression alphabet** of S is S plus a bunch of extra symbols; it's S ∪ {<span class="code">(</span>, <span class="code">)</span>, <span class="code">\*</span>, <span class="code">∪</span>, <span class="code">Ø</span>} (I assume that none of those symbols are already in S.)

I'm doing something that I said earlier that I would try not to do. I'm using symbols in an *alphabet* that I also use in *expressions* that talk about strings in that alphabet. (Of course, I also said that this would all fall apart when we got to regular expressions, and sure enough, it did. Foreshadowing: the sign of a quality blog.)

Again, keep a careful eye on when I'm using fixed-width blue, because those are the "meaningless" symbols, not expression notation.

**Definition 10**: Take any alphabet S. The **regular expression language** R of an alphabet S is a language formed from strings of the regular expression language of S, and is defined as follows:

  - <span class="code">Ø</span> is in R.
  - Every member of S is in R
  - If u and w are strings in R then <span class="code">(</span>uw<span class="code">)</span> is in R.
  - Similarly, <span class="code">(</span>u<span class="code">∪</span>w<span class="code">)</span> is in R.
  - Similarly, u<span class="code">\*</span> is in R.
  - Nothing else is in R

An example might help. Suppose that S = {<span class="code">a</span>, <span class="code">b</span>}. The regular expression alphabet of S is {<span class="code">a</span>, <span class="code">b</span>, <span class="code">(</span>, <span class="code">)</span>, <span class="code">\*</span>, <span class="code">∪</span>, <span class="code">Ø</span>}. The regular expression language of S is R = {<span class="code">Ø</span>, <span class="code">a</span>, <span class="code">b</span>, <span class="code">(ØØ)</span>, <span class="code">(Øa)</span>, <span class="code">(Øb)</span>, <span class="code">(aØ)</span>, <span class="code">(aa)</span>, <span class="code">(ab)</span>, <span class="code">(bØ)</span>, <span class="code">(ba)</span>, <span class="code">(bb)</span>, <span class="code">(Ø∪Ø)</span>, <span class="code">(Ø∪a)</span>, <span class="code">(Ø∪b)</span>, <span class="code">(a∪Ø)</span>, <span class="code">(a∪a)</span>, <span class="code">(a∪b)</span>, <span class="code">(b∪Ø)</span>, <span class="code">(b∪a)</span>, <span class="code">(b∪b)</span>, <span class="code">Ø\*</span>, <span class="code">a\*</span>, <span class="code">b\*</span>, …}

We've defined an alphabet, we've defined a language -- a language which looks suspiciously like the expressions we've been using to talk *about languages*. Next time we'll do something insanely clever to bridge the gap between strings in the regular expression language and the languages which these strings would define if they were interpreted as expressions.

</div>

</div>

</div>

