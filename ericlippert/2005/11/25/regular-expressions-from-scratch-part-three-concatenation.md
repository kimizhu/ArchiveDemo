<div id="page">

# Regular Expressions From Scratch, Part Three: Concatenation

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/25/2005 10:00:00 AM

-----

<div id="content">

<div class="mine">

You probably intuitively understood concatenation already, but let me define it anyway.

**Definition 5**: The **concatenation** of two strings w and u (over the same alphabet) makes a string consisting of the sequence of every element in w followed by every element in u. We write concatenations the same way as before: all run together. So if we have S = { <span class="code">a</span>, <span class="code">b</span>, <span class="code">c</span>} and two strings in S\* called w and u, where w=<span class="code">abc</span> and u=<span class="code">cccaaabbb</span> then then wu=<span class="code">abccccaaabbb</span>.

Be very clear on this point: w and u are *variables* which represent string values. wu is an expression which represents concatenating the value of u onto the end of the value of w. Similarly, w<span class="code">aaaa</span> would be the result of concatenating <span class="code">aaaa</span> ont w, so w<span class="code">aaaa</span> = <span class="code">abcaaaa</span>.

Recall that I'm using the variable e to represent the empty string. You can concatenate the empty string onto any old string, and the result is unchanged. So we = w, for any string w.

We can concatenate any *finite* number of strings together with impunity. If v=<span class="code">ab</span> then wuvev<span class="code">a</span>e<span class="code">a</span>e<span class="code">a</span> = <span class="code">abccccaaabbbababaaa</span>.

We cannot concatenate an *infinite* number of non-empty strings together to form a string, because strings are by definition finite sequences.

Note that with this definition we can say that the set S\* is the set of *all finite concatenations* of any members of S, plus the empty string.

That pretty much takes care of making strings out of other strings. We're going to need to make new languages from old languages as well. Can we meaningfully concatenate two *languages* together? Sure, why not?

**Definition 6**: Two languages L and K over the same alphabet may be concatenated together form a new language. Concatenated languages are notated as you'd expect: by writing one after the other. If u is in L, and v is in K, then uv is in LK.

If one of those languages is the empty language - the language with no strings, not even the empty string - then the concatenation is also the empty language.

For example, suppose

S = {<span class="code">a</span>, <span class="code">b</span>}  
L = {e, <span class="code">a</span>, <span class="code">aa</span>, <span class="code">aaa</span>, <span class="code">aaaa</span>, …}  
K = {e, <span class="code">b</span>, <span class="code">bb</span>, <span class="code">bbb</span>, <span class="code">bbbb</span>, …}

then

LK = {w in S\* such that w has any number of <span class="code">a</span>s followed by any number of <span class="code">b</span>s}

That's a little irksome. We need a more compact notation for "any number of <span class="code">a</span>s followed by any number of <span class="code">b</span>s". Fortunately we already have such a notation -- we have the Kleene Closure over alphabets. We'll just create two one-character alphabets, star them to form languages, and concatenate the languages:

LK = {<span class="code">a</span>}\*{<span class="code">b</span>}\*

Let's make that notation a little more compact. From now on when I say

X = <span class="code">a</span>\*

what I mean is

X = {<span class="code">a</span>}\*

That is, I'm going to omit the set braces, because you can figure them out from context.

We can write that last example much more compactly:

S = {<span class="code">a</span>, <span class="code">b</span>}  
L = <span class="code">a</span>\*  
K = <span class="code">b</span>\*  
LK = <span class="code">a</span>\*<span class="code">b</span>\*

Perhaps you can see that we are in fact heading towards regular expressions.

Next time: The Kleene Closure applies to languages too.

</div>

</div>

</div>

