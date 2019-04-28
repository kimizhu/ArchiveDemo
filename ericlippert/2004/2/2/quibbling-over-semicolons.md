<div id="page">

# Quibbling Over Semicolons

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/2/2004 4:55:00 PM

-----

<div id="content">

<div class="mine">

In a comment to my last entry Dan Shappir said:

*<span style="color: #000080;">In C/C++/Java/C\# statements must end with a semicolon. JavaScript OTOH allows the use of a newline as a statement separator. Presumably this was done so that it would be easier to use for scripters. Personally, I hate the ambiguities it generates. </span>*

I'm with you Dan -- I find it irritating as well.  But we're stuck with it now.

Let me clarify a bit what is actually going on here. Thinking that a newline is a legal statement separator in JScript is actually not the best way to think about this language feature. A better way to think about it is that **JScript requires some statements to end in a semicolon but the parser will automatically insert missing semicolons.** The net result of both statements is the same, but I find it easier to think about semicolons as required and the compiler as automatically fixing some mistakes.

First off, what statements require semicolons? All the following require a semicolon:

  - an empty statement - a single semicolon is a legal statement
  - a var declaration
  - an expression evaluation statement
  - a do-while loop
  - a continue statement
  - a break statement
  - a return statement
  - a throw statement

The automatic semicolon inserter scans the program looking for places that semicolons are required but missing. It automatically inserts the semicolon provided that these three conditions are met:

<div>

</div>

  - The missing semicolon goes before a newline, a right-curly-brace, or the end of the program.
  - Adding a missing semicolon does not create an empty statement.
  - Adding a missing semicolon does not screw up the arguments to the "for" loop. (See the spec for the exact details.)

This leads to a few bizarre situations, because programs like these are now hard to parse:

<span class="code"> </span>

a  
\++  
b

Is that <span class="code">a++; b;</span> or <span class="code">a; ++b;</span> ?

To disambiguate this, **JScript restricts where newlines can go.** You can't put a newline in the middle of a ++, --, return, throw, break or continue. (Recall that JScript supports *labelled* break and continue.)

For example, the automatic semicolon inserter turns this:

<span class="code"> </span>

return  
a++

into

<span class="code"> </span>

return;  
a++;

Which means that

<span class="code"> </span>

if (b) return  
M()

means

<span class="code"> </span>

if (b) return;  
M();

and not

<span class="code"> </span>

if (b) return M();

Which is great, unless of course the latter is what you *wanted*.

Auto semi insertion can bite you in many ways. Consider for example:

<span class="code"> </span>

Number.prototype.blah = function(){ /\* whatever \*/ }  
var d = 1, e = 2  
var a = d \* e  
(d + e).blah()

Auto semicolon insertion turns that into

<span class="code"> </span>

Number.prototype.blah = function(){ /\* whatever \*/ };  
var d = 1, e = 2;  
var a = d \* e // no semi\!  
(d + e).blah();

because of course <span class="code">var a = d \* e(d + e).blah();</span> is perfectly legal.  Nonsensical at runtime but syntactically legal.

Dan brings up another interesting case in the comments to this post; what about <span class="code">{foo:bar()}</span> ? Is this a statement that consists of just an anonymous object expression with a single member foo equal to the value returned by bar(), so <span class="code">{foo:bar()};</span>? Or is this a code block with a labeled call to bar(), so <span class="code">{foo:bar();}</span>?

It is the latter; the specification already makes it illegal to have a statement which consists of a single expression that begins with a left curly, because obviously allowind that would make it difficult to disambiguate between a block of statements and an anonymous object. The automatic semicolon inserter never even considers the possibility that this might be an illegal program that is also missing semicolons.

My advice is to **use semicolons rather than relying upon the crazy rules for the auto semi inserter.**

</div>

</div>

</div>

