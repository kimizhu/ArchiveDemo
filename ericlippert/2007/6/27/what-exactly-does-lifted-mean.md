<div id="page">

# What exactly does 'lifted' mean?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/27/2007 10:00:00 AM

-----

<div id="content">

<div class="mine">

(Note: all type placeholders <span class="code">S</span>, <span class="code">T</span>, <span class="code">U</span> that I mention in this article are non-nullable value types.)

I got a question the other day from a reader who had been taking a close look at the C\# 2.0 standard. He noticed that when we talk about nullables in the standard we talk about "lifting", but we do so inconsistently. To summarize what the standard says:

1\) For every equality and relational operator that compares an <span class="code">S</span> to a <span class="code">T</span> and returns a <span class="code">bool</span> there is a corresponding **lifted** operator that compares an <span class="code">S?</span> to a <span class="code">T?</span> and returns a <span class="code">bool. </span>It returns <span class="code">false</span> if either argument is <span class="code">null</span>. (With the additional exception that if one argument to an equality operator is the null literal then the result may be determined by checking if the nullable term has a value.)

<span class="code"></span>2) For the unary operators <span class="code">+ ++ - -- \! \~</span> where the operator takes an <span class="code">S</span> and returns a <span class="code">T</span> there is a corresponding **lifted** operator that takes an <span class="code">S?</span> and returns a <span class="code">T?</span>. It returns <span class="code">null</span> if its argument is <span class="code">null</span>.

3\) For the binary operators <span class="code">+ - \* / % & | ^ \<\< \>\></span>, where the operator takes an <span class="code">S</span> and a <span class="code">T</span> and returns a <span class="code">U</span> there is a corresponding **lifted** operator that takes an <span class="code">S?</span> and a <span class="code">T?</span> and returns a <span class="code">U?</span>. It returns <span class="code">null</span> if either argument is <span class="code">null</span>.

4\) For user-defined conversions from <span class="code">S</span> to <span class="code">T</span>, there is a corresponding **lifted** user-defined conversion from <span class="code">S?</span> to <span class="code">T?</span>. It returns <span class="code">null</span> if the argument is <span class="code">null</span>.

5\) For built-in conversions from <span class="code">S</span> to <span class="code">T</span>, there are corresponding **nullable** conversions:  
5.1) from <span class="code">S?</span> to <span class="code">T?</span>, which returns <span class="code">null</span> if the argument is <span class="code">null</span>.  
5.2) from <span class="code">S</span> to <span class="code">T?</span>, which never returns <span class="code">null</span>, and  
5.3) an explicit-only conversion from <span class="code">S?</span> to <span class="code">T</span> which throws if the argument is <span class="code">null</span>.

Why does the standard call out that the conversions in (5) are "nullable" but not "lifted"? Everything else is "lifted"\!

This is a bit of a mess I'm afraid.

I talked to [Dr. T](http://blogs.msdn.com/madst/default.aspx) (AKA Mads Torgersen, the C\# language PM) about this. He defined for me precisely what mathematicians mean by “lifted”.

Suppose we've got a function <span class="math">f</span> which maps values from a set <span class="math">A</span> into a set <span class="math">B</span>. That is <span class="math">f:A→B</span>.

Suppose further that <span class="code">null</span> is not a member of either <span class="math">A</span> or <span class="math">B</span>.

Now consider the sets <span class="math">A' = A ∪ { null }</span> and <span class="math">B' = B ∪ { null }</span>

We define the "lifted function" <span class="math">f'</span> as

<span class="math">f':A'→B'</span> such that <span class="math">f'(x) = f(x)</span> for all <span class="math">x ∈ A </span>and <span class="math">f'(null) = null</span>

Similarly, if we had a two-argument function <span class="math">f: A × B → C</span>, we would define <span class="math">f': A' × B' → C'</span> as <span class="math">f'(x,y) = f(x,y)</span> for all <span class="math">(x,y) ∈ A × B</span> and <span class="math">null</span> if either <span class="math">x</span> or <span class="math">y</span> is <span class="math">null</span>.

What we’re getting at here is that “lifted” means “takes nulls, always agrees with the unlifted version when arguments are not null, maps everything else onto null”.

Now you probably see why I said this was a bit of a mess. By the mathematician's definition, (1) is incorrectly called "lifted", (2), (3), and (4) are correctly called "lifted", (5.1) is incorrectly NOT called "lifted", and (5.2) and (5.3) are correctly not called "lifted". Of course we can choose to define what "lifted" means *in C\#* any way we like, but it would be nice if our definition agreed with convention and was used consistently\!

I regret the confusion. I do not believe there is any particular sensible reason for these inconsistencies. Rather, the details were changed so many times over the years as the nullable feature was developed that these sorts of subtle problems crept into the spec and were never expunged. Though of course all of us have as a goal that the standard be a model of clarity and permanence, it is fundamentally a working, evolving, imperfect document; these kinds of things will happen. Hopefully in the next version of the standard some of these sorts of details will be tidied up.

I hope that answers the question\!  

</div>

</div>

</div>

