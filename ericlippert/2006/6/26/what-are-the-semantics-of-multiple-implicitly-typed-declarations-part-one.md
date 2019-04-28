<div id="page">

# What Are The Semantics Of Multiple Implicitly Typed Declarations? Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/26/2006 2:02:00 PM

-----

<div id="content">

<div class="mine">

In my earlier series on inferring a unique "best" type from a set of expressions I mentioned that one potential application of such an algorithm is in implicitly typed variables. This led to some good questions and concerns posted in the comments - questions and concerns which echo similar feedback we've been receiving from a variety of sources since we released the first technology preview of C\# 3.0 last year.

I'd like to run a quick unscientific poll to see what your intuitions and expectations about implicitly typed variables with multiple declarations are. Please leave a comment describing what you think should happen here, and why you think that.

1: local variable declaration <span class="code">var x = 1, y = 2.0;</span> has the same semantics as:  
(a) <span class="code">double x = 1, y = 2.0; </span>  
(b) <span class="code">int x = 1; double y = 2.0; </span>  
(c) <span class="code">object x = 1, y = 2.0; </span>  
(d) this should be a compile-time error  
(e) something else, please specify  

2: local variable declaration <span class="code">var q = 0, r = (short)6;</span> has the same semantics as:  
(a) <span class="code">int q = 0; short r = 6; </span>  
(b) <span class="code">int q = 0, r = 6; </span>  
(c) <span class="code">short q = 0, r = 6; </span>  
(d) <span class="code">object q = 0, r = 6; </span>  
(e) this should be a compile-time error  
(f) something else, please specify  

Thanks\! Next time I'll describe some of the pros and cons of each and what our current thinking is in this area.

</div>

</div>

</div>

