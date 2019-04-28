# JScript eval redux, and some spec diving

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/22/2003 10:44:00 PM

-----

I was discussing the difference between executing in local and global scopes the other day.  A reader points out something that I forgot to mention – there are two sneaky ways to manipulate the global namespace from an “eval” in Jscript. 

 

 

First, the Function constructor constructs a named function in the global scope. This had slipped my mind when I was writing the entry.

 

 

The second trick was very much on my mind but I did not mention as it would be yet another digression.  That is the fact that assigning a value to an undeclared variable creates a new variable in global scope.

 

 

This was on my mind because a couple weeks ago my buddy (and open source zealot, I mean *enthusiast*) CJ was debugging an irksome incompatibility between Gecko and IE. It turned out to hinge on the fact that in IE, fetching the value of an undefined variable is illegal, but setting it is legal.  According to CJ, in Gecko both are legal.  (I wouldn’t know, never having actually used any browser other than IE since IE3 was in development.)

 

 

Now, if you look at the ECMAScript Revision 3 specification (aka E3) in some depth it becomes clear that IE and Gecko are both in compliance with the spec, and yet incompatible with each other.  

"Howzat?" I hear you ask. The logic is a little tortuous\!

 

 

Creating a new global variable when setting an undeclared variable must be legal according to E3 section 10.1.4, line 5, which states that an identifier undeclared in all scopes on the scope chain results in a "null reference", and section 8.7.2, line 6 which states that an assignment to a null reference creates a new variable in the global scope.   IE does this, and I assume that Gecko does as well.

 

 

But setting the value of an undeclared variable must throw an error according to E3 section 8.7.1, line 3, which states that fetching the value of a null reference creates a ReferenceError exception. IE does this. If Gecko creates a variable in some scope rather than throwing a ReferenceError exception then clearly they have produced a situation in which a program running in Gecko has different semantics than when running in the browser used by the other 90% of the world.  

 

Such situations are, as my buddy CJ discovered, very painful for developers -- mitigating this pain is why my colleagues and I went to the massive trouble and expense of defining the specification in the first place\!  However, if that is the case then Gecko is not \_actually\_ in violation of the specification thanks to E3 section 16, which states:

 

*"An implementation may provide additional types, values, objects, properties, and functions beyond those described in this specification. This may cause constructs (**such as looking up a variable in the global scope**) to have implementation-defined behaviour instead of throwing an error (**such as ReferenceError**)." * \[Emphasis added\]* *

 

 

The E3 authors explicitly added the parenthetical clauses to make Gecko-like behaviour legal, though discouraged.  However, the clause is necessary -- without this clause it becomes very difficult to define certain browser-object-model/script-engine interactions in a manner which does not (a) make both IE and Navigator technically noncompliant with the spec, (b) drag lots of extra-language browser semantics into the language specification and (c) make it difficult to extend the language in the future.  

 

 

We earnestly wished to avoid all these situations, so the rule became "any error situation may legally have non-error semantics." This is in marked contrast to, say, the ANSI C specification which rigidly defines what error messages a compliant implementation must produce under various circumstances.

