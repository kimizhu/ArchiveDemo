# Lambda Expressions vs. Anonymous Methods, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/11/2007 3:40:00 PM

-----

We interrupt the discussion of how the difference between lambda expression and anonymous method convertibility leads to potential performance problems in the compiler to answer a user question.

Within hours of yesterday's post readers [Raymond Chen](http://blogs.msdn.com/oldnewthing) and [Marc Brooks](http://musingmarc.blogspot.com/) both asked me the same question. **How do lambdas and anonymous methods interact with implicitly typed local variables?**

Recall that C\# 3.0 will support implicitly typed local variables. This:

var d = new Dictionary\<string, List\<int\>\>(); 

is a syntactic sugar for

Dictionary\<string, List\<int\>\> d = new Dictionary\<string, List\<int\>\>(); 

Again, I wish to emphasize that [C\# 3.0 is still statically typed, honest\!](http://blogs.msdn.com/ericlippert/archive/2005/09/27/474462.aspx) The var keyword does not have the semantics of the JScript var or VBScript's Variant or any such thing. It simply means "the variable is the type of the right side of the declaration". We are committed to keeping C\# 3.0 a [statically typed](http://blogs.msdn.com/wesdyer/archive/2006/12/20/types-of-confusion.aspx) language.

You will note that a key requirement there is that the right hand side actually has a type. Furthermore, it cannot be the null type or the void type, obviously. So then what should this do?

var f = i=\>F(i); 

Clearly this cannot be legal; since as I discussed [yesterday](http://blogs.msdn.com/ericlippert/archive/2007/01/10/lambda-expressions-vs-anonymous-methods-part-one.aspx) the type of the formal parameter is determined from the target type, and we are trying to determine the target type, we have a chicken-and-egg problem where there is not enough information to determine the type.

What about this?

var f = (int i)=\>i; 

Here we know the type of the parameter, we can infer the type of the return easily enough, it's the same type as the parameter\! We're set, right?

Not so fast.

Consider the following type declarations:

public delegate int D1(int i);  
public delegate int D2(int i);  
public delegate int D3\<A\>(A a);  
public delegate R D4\<A, R\>(A a);

If the var were replaced by D1, D2, D3\<int\>, D4\<int, int\>, or for that matter, D4\<int, double\> or D4\<int, object\>, we'd have a legal program. So what *is* the type of the right hand side? How can we choose which of these is the best?

We can't. Delegate types are not even structurally equivalent; you can't even assign a variable of type D1 to a variable of type D2. We would always potentially choose wrong.

What this tells us is that it is more than just that the formal parameter types of an implicitly typed lambda expression flow from the target type. The type of the *entire* lambda expression flows from the target type, and therefore, **lambda expressions themselves must have no type**.

And in fact the C\# 2.0 specification calls this out. Method group expressions and anonymous method expressions are *typeless expressions* in C\# 2.0, and lambda expressions join them in C\# 3.0. Therefore it is illegal for them to appear "naked" on the right hand side of an implicit declaration.

This is rather unfortunate because it makes it difficult to declare a variable of type "delegate which takes or returns an anonymous type". The whole point of the var feature in the first place is to make anonymous types work, and yet we seem to have a hole here. As it turns out, it is possible\! Next time I'll describe a clever trick whereby we can use *method type inference* to make variable type inference work correctly on lambdas which take anonymous types.

