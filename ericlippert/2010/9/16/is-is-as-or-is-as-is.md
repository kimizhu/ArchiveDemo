<div id="page">

# Is is as or is as is?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2010 6:33:00 AM

-----

<div id="content">

<div class="mine">

Today a question about the <span class="code">is</span> and <span class="code">as</span> operators: is the <span class="code">is</span> operator implemented as a syntactic sugar for the <span class="code">as</span> operator, or is the <span class="code">as</span> operator implemented as a syntactic sugar for the <span class="code">is</span> operator? More briefly, is <span class="code">is as</span> or is <span class="code">as is</span>?

Perhaps some sample code would be more clear. This code

<span class="code"> </span>

bool b = x is Foo;

could be considered as a syntactic sugar for

<span class="code"> </span>

bool b = (x as Foo) \!= null;

in which case <span class="code">is</span> is a syntactic sugar for <span class="code">as</span>. Similarly,

<span class="code"> </span>

Foo f = x as Foo;

could be considered to be a syntactic sugar for

<span class="code"> </span>

var temp = x;  
Foo f = (temp is Foo) ? (Foo)temp : (Foo)null;

in which case <span class="code">as</span> is a syntactic sugar for <span class="code">is</span>. Clearly we cannot have both of these be sugars because then we have an infinite regress\!

The specification is clear on this point; <span class="code">as</span> (in the non-dynamic case) is defined as a syntactic sugar for <span class="code">is</span>.

However, in practice the CLR provides us instruction <span class="code">isinst</span>, which ironically acts like <span class="code">as</span>. Therefore we have an instruction which implements the semantics of <span class="code">as</span> pretty well, from which we can build an implementation of <span class="code">is</span>. In short, *de jure* <span class="code">is</span> is <span class="code">is</span>, and <span class="code">as</span> is as <span class="code">is</span> is, but *de facto* <span class="code">is</span> is <span class="code">as</span> and <span class="code">as</span> is <span class="code">isinst</span>.

I now invite you to leave the obvious jokes about President Clinton in the comments.

</div>

</div>

</div>

