# Bad Recursion Revisited

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/28/2005 12:12:00 PM

-----

We have internal email lists for questions about programming languages. Here's one that came across recently that I thought illustrated a good point about language design.

An interview candidate gave the following awful implementation of the factorial function. (Recall that factorial is notated "n\!", and is defined as the product of all the integers from 1 to n. 0\! is defined as 1. So 4\! = 4 x 3 x 2 x 1 = 24.)

If you note that n\! = n x ((n-1)\!) then a [recursive solution](http://blogs.msdn.com/ericlippert/archive/2004/05/19/135392.aspx) comes to mind. When asked to implement the factorial function in C, an interview candidate came up with this:

 

int F(int x){  
  return (x \> 1) ? (x \* F(--x)) : x;  
}

Now, leaving aside for the moment the fact that [this badly-named function does no bounds checking on the inputs, potentially consumes the entire stack, returns a wrong or nonsensical answer for inputs less than one, and is a recursive solution to a problem that can easily be solved with a simple lookup table](http://blogs.msdn.com/ericlippert/archive/2004/05/20/results-of-the-fibonacci-challenge-are-in.aspx), it has another big problem -- it doesn't even return the correct answer for any input greater than one either\! F(4) will return 6, not 24, in Microsoft C.

And yet the *seemingly* equivalent C\#, JScript and VBScript programs return 24.

The question was "What the heck is up with that?"

This looks perfectly straightforward. If x is 4, then we evaluate the consequence of the conditional operator...  
  
4 \* F(3)  
\= 4 \* 3 \* F(2)  
\= 4 \* 3 \* 2 \* F(1)  
\= 4 \* 3 \* 2 \* 1  
\= 24

right? What is broken with C?

Page 52 of K\&R has the answer.

 

"C, like most languages, does not specify the order in which the operands of an operator are evaluated. (The exceptions are &&, ||, ?: and ','.) For example, in a statement like x = f()+g(); f may be evaluated before g or vice versa; thus if either f or g alters a variable on which the other depends, x can depend on the order of evaluation."

The Microsoft C compiler actually evaluates the function call first, which causes x to be decremented *before* the multiplication. So this is actually the same as  
  
3 \* F(3)  
\= 3 \* 2 \* F(1)  
\= 3 \* 2 \* 1  
\= 6.

Why does the specification call out that the compiler can choose any order for subexpression evaluation? Because then the compiler can choose to optimize the order so that it consumes a small number of stack or register slots for the results.

Of course, C was written back in the dark ages -- the 1970's -- when indeed most languages were pretty ill-specified and full of terrible "gotchas" like this. JScript, VBScript, C\#, and most modern languages do guarantee that functions will be evaluated in left-to-right order. In all these languages,

x = f() + g() \* h();

will evaluate f, then g, then h.

As K\&R notes "The moral is that writing code that depends on order of evaluations is a bad programming practice in any language." Amen to that\!

