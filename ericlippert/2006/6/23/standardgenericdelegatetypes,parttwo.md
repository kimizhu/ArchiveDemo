# Standard Generic Delegate Types, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/23/2006 1:18:00 PM

-----

[Last time](http://blogs.msdn.com/ericlippert/archive/2006/06/21/641831.aspx) I asked whether there were examples of delegate types which could be declared with the traditional delegate type declaration syntax that could not be declared with the generic syntax, and Alois Kraus came up with a number of examples. This syntax does not support definition of void delegates, delegates with out or ref parameters, delegates with parameter arrays, generic delegates, delegates which have unsafe parameter or return types, and delegates declarations with attributes on the parameters.

However, there is one other difference between the traditional and generic declaration forms that no one mentioned: the traditional syntax adds a new symbol to the scope, and that symbol can be used in a recursive definition. The generic syntax does not add a new symbol to the scope, so you cannot define a recursively defined delegate type with it.

Consider, for example, the "purest" possible delegate type -- a function that takes a function and returns a function, where both the argument and the return are themselves functions that take functions and return functions. Such a function is called a *combinator*. That is:

delegate D D(D d); 

Using the new lambda syntax, you could concisely define all kinds of combinators:

D I = x=\>x  
D M = x=\>x(x);  
D K = x=\>y=\>x;  
D L = x=\>y=\>x(y(y));  
D S = x=\>y=\>z=\>(x(z(y(z))));  
D C = a=\>b=\>c=\>a(c)(b);  
D B = a=\>b=\>c=\>a(c(b));  

Those of you with a background in combinatory logic will certainly recognize all of these combinators. They're of considerable theoretical interest, but not much practical use in C\#. (I might discuss some of the properties of these combinators in a later posting, but if you're interested this fascinating subject, I recommend reading Raymond Smullyan's delightful book of combinatory logic puzzles "To Mock A Mockingbird".)

There's no way to define a delegate signature for a combinator using only the generic syntax because it goes into an infinite regress immediately. Neat, eh?

