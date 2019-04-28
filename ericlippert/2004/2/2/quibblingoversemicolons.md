# Quibbling Over Semicolons

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/2/2004 4:55:00 PM

-----

In a comment to my last entry Dan Shappir said:

*In C/C++/Java/C\# statements must end with a semicolon. JavaScript OTOH allows the use of a newline as a statement separator. Presumably this was done so that it would be easier to use for scripters. Personally, I hate the ambiguities it generates. *

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

  - The missing semicolon goes before a newline, a right-curly-brace, or the end of the program.
  - Adding a missing semicolon does not create an empty statement.
  - Adding a missing semicolon does not screw up the arguments to the "for" loop. (See the spec for the exact details.)

This leads to a few bizarre situations, because programs like these are now hard to parse:

 

a  
\++  
b

Is that a++; b; or a; ++b; ?

To disambiguate this, **JScript restricts where newlines can go.** You can't put a newline in the middle of a ++, --, return, throw, break or continue. (Recall that JScript supports *labelled* break and continue.)

For example, the automatic semicolon inserter turns this:

 

return  
a++

into

 

return;  
a++;

Which means that

 

if (b) return  
M()

means

 

if (b) return;  
M();

and not

 

if (b) return M();

Which is great, unless of course the latter is what you *wanted*.

Auto semi insertion can bite you in many ways. Consider for example:

 

Number.prototype.blah = function(){ /\* whatever \*/ }  
var d = 1, e = 2  
var a = d \* e  
(d + e).blah()

Auto semicolon insertion turns that into

 

Number.prototype.blah = function(){ /\* whatever \*/ };  
var d = 1, e = 2;  
var a = d \* e // no semi\!  
(d + e).blah();

because of course var a = d \* e(d + e).blah(); is perfectly legal.  Nonsensical at runtime but syntactically legal.

Dan brings up another interesting case in the comments to this post; what about {foo:bar()} ? Is this a statement that consists of just an anonymous object expression with a single member foo equal to the value returned by bar(), so {foo:bar()};? Or is this a code block with a labeled call to bar(), so {foo:bar();}?

It is the latter; the specification already makes it illegal to have a statement which consists of a single expression that begins with a left curly, because obviously allowind that would make it difficult to disambiguate between a block of statements and an anonymous object. The automatic semicolon inserter never even considers the possibility that this might be an illegal program that is also missing semicolons.

My advice is to **use semicolons rather than relying upon the crazy rules for the auto semi inserter.**

