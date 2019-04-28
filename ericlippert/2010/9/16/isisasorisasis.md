# Is is as or is as is?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2010 6:33:00 AM

-----

Today a question about the is and as operators: is the is operator implemented as a syntactic sugar for the as operator, or is the as operator implemented as a syntactic sugar for the is operator? More briefly, is is as or is as is?

Perhaps some sample code would be more clear. This code

 

bool b = x is Foo;

could be considered as a syntactic sugar for

 

bool b = (x as Foo) \!= null;

in which case is is a syntactic sugar for as. Similarly,

 

Foo f = x as Foo;

could be considered to be a syntactic sugar for

 

var temp = x;  
Foo f = (temp is Foo) ? (Foo)temp : (Foo)null;

in which case as is a syntactic sugar for is. Clearly we cannot have both of these be sugars because then we have an infinite regress\!

The specification is clear on this point; as (in the non-dynamic case) is defined as a syntactic sugar for is.

However, in practice the CLR provides us instruction isinst, which ironically acts like as. Therefore we have an instruction which implements the semantics of as pretty well, from which we can build an implementation of is. In short, *de jure* is is is, and as is as is is, but *de facto* is is as and as is isinst.

I now invite you to leave the obvious jokes about President Clinton in the comments.

