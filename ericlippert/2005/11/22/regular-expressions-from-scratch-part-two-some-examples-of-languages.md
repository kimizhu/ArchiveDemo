<div id="page">

# Regular Expressions From Scratch, Part Two: Some Examples of Languages

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/22/2005 10:00:00 AM

-----

<div id="content">

<div class="mine">

Let's look at some sample languages to get a sense of just how flexible languages can be.

For example, here are some languages over the alphabet S = {<span class="code">0</span>, <span class="code">1</span>}

L<sub>1</sub> = all members of S\*, period -- the language where every string is in the language

L<sub>2</sub> = all members of S\* such that there are more <span class="code">1</span>'s than <span class="code">0</span>'s. So L<sub>2</sub> = {<span class="code">1</span>, <span class="code">11</span>, <span class="code">011</span>, <span class="code">101</span>, <span class="code">110</span>, <span class="code">111</span>, …}

L<sub>3</sub> = all members of S\* such that the string is a prime number of <span class="code">1</span>'s. So L<sub>3</sub> = {<span class="code">11</span>, <span class="code">111</span>, <span class="code">11111</span>, <span class="code">1111111</span>, <span class="code">11111111111</span>, …}

L<sub>4</sub> = all members of S\* such that there are n <span class="code">1</span>'s and no <span class="code">0</span>'s in the string, and furthermore, n is such that there is a nontrivial solution to Fermat's Last Theorem: a<sup>n</sup> + b<sup>n</sup> = c<sup>n</sup> in natural numbers.

That is a perfectly good description of a language. In fact, it's a description of the finite language L<sub>4</sub> = {<span class="code">1</span>, <span class="code">11</span>}. We know that now, as Fermat's Last Theorem was proven. Twenty years ago though we would not have been able to enumerate all the members of this language with any confidence.

We can even come up with language descriptions that we cannot possibly know their members\!

L<sub>5</sub> = all members of S\* such that there are n <span class="code">1</span>'s and no <span class="code">0</span>'s in the string, and furthermore, n is the largest number of quarters I had in my pocket at any time on the 13th of December, 1991.

That language has only one member, but darned if I know what it is\!

This silly example raises an important point: in general, given a description of a language, we cannot know how many members the language has, whether it is finite or infinite, or whether any particular string is a member of the language.

We will be seeing later that there are definitely languages which cannot *in principle* be determined. That is, there is no way whatsoever of computing the members of the language.

For some languages though, we can build and analyze devices which recognize them - that is, when given a string and a language, a device can tell you whether or not the string is a member of the language.

To do that, we're going to need a way to describe languages in a very precise manner - we need a "language definition language" of some sort. The relationship between language recognizers and various metalanguages for describing languages will be a fundamental focus of this series.

Next time: concatenation of strings and languages

</div>

</div>

</div>

