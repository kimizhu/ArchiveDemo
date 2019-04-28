<div id="page">

# Regular Expressions From Scratch, Part Six: The Insanely Clever Bit

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/5/2005 10:10:00 AM

-----

<div id="content">

<div class="mine">

Let's start with an easy one today, because things are about to get a little tricky.

**Definition 10**: a **pair** is a finite sequence with exactly two members.

**Definition 11**: a **function** is a set of pairs, where the second member of each pair is the value associated with the first member by the function. If F is a function { (a1, b1), (a2, b2) } then we might write F(a1) = b1, F(a2) = b2. (Again, we will figure out from context whether parentheses mean function mapping, grouping or a finite sequence.)

**Definition 12**: Take any alphabet S. The **regular expression language generating function** for S is a function where the first member of the pair is a string in the regular expression language of S, and the second member is a language in S.

Let's call the function L and the regular expression language R. The pairs in this set go like this.

  - The string <span class="code">Ø</span> is paired with the empty language. Or, in our new notation, L(<span class="code">Ø</span>) = {}
  - The strings in R that are just single members of S are paired with the languages that are just single members of S. For example, if S = {<span class="code">a</span>, <span class="code">b</span>} then L(<span class="code">a</span>) = {<span class="code">a</span>}, L(<span class="code">b</span>) = {<span class="code">b</span>},
  - If x and w are in R then L(<span class="code">(</span>xw<span class="code">)</span>) = L(x)L(w)
  - Similarly, L(<span class="code">(</span>x<span class="code">∪</span>w<span class="code">)</span>) = L(x) ∪ L(w)
  - Similarly, L(x<span class="code">\*</span>) = L(x)\*

OK, let's try it out. What's L(<span class="code">((ab)∪a\*)</span>)?

L(<span class="code">((ab)∪a\*)</span>)  
\= L(<span class="code">(ab)</span>) ∪ L(<span class="code">a\*</span>)  
\= L(<span class="code">a</span>)L(<span class="code">b</span>) ∪ L(<span class="code">a\*</span>)  
\= {<span class="code">a</span>}{<span class="code">b</span>} ∪ L(<span class="code">a</span>)\*  
\= {<span class="code">a</span>}{<span class="code">b</span>} ∪ {<span class="code">a</span>}\*  
\= ((<span class="code">ab</span>) ∪ <span class="code">a</span>\*)  
\= {<span class="code">ab</span>, e, <span class="code">a</span>, <span class="code">aa</span>, <span class="code">aaa</span>, <span class="code">aaaa</span>, …}

Hey, wouldja look at that: L(<span class="code">((ab)∪a\*)</span>) = ((<span class="code">ab</span>) ∪ <span class="code">a</span>\*)

What an amazing coincidence\!

Obviously this is no coincidence at all. We have just defined a mapping between regular expressions and the languages which consist of all strings which match those regular expressions, and the mapping is basically "turn the regular expression string into an expression via the obvious substitutions."

Since we have this very strong mapping, I am probably going to become very sloppy about making a distinction between L(<span class="code">((ab)∪a\*)</span>) and ((<span class="code">ab</span> ∪ <span class="code">a</span>\*). If I say "the language <span class="code">((ab)∪a\*)</span> what I mean is L(<span class="code">((ab)∪a\*)</span>).

Note also that L(<span class="code">((ab)a)</span>) = L(<span class="code">(a(ba))</span>) = { {<span class="code">aba</span>}. In general, the concatenation and union operators do not require parens. Therefore, from now on I will also be very sloppy with my parens. Even though <span class="code">aba</span> is not a "real" regular expression by the rules laid out earlier, I will assume that you can mentally transform that back into the well-formed <span class="code">(a(ba))</span> string.

Let's end off today's descent into the bowels of computer science with a rather obvious definition:

**Definition 13**: A language K over an alphabet S is called a **regular language** if and only if there exists a string r in S's regular expression language such that L(r) = K. That is, a language is called a "regular language" if and only if it can be described by a regular expression.

Is *every* language regular? One would suspect not, given the incredible variety of languages that I mentioned earlier. Actually proving that a nonregular language exists is both amusing and character-building, so we'll do that next.

</div>

</div>

</div>

