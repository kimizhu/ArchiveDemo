# Ignoring parentheses

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/12/2010 6:53:00 AM

-----

Yet another [amusing question from StackOverflow](http://stackoverflow.com/questions/2186595/c-is-there-a-difference-between-return-myvar-vs-return-myvar): is there a difference between “return something;” and “return (something);” in C\#? In *practice*, there is no difference. In *theory* there could be a difference. There are three interesting points in the C\# specification where this could present a problem. First, conversion of anonymous functions to delegate types and expression trees. Consider the following:  

Func\<int\> F1() { return ()=\>1; }  
Func\<int\> F2() { return (()=\>1); }

F1 is clearly legal. Is F2? Technically, no. The spec says in section 6.5 that there is a conversion from a lambda expression to a compatible delegate type. Is that a *lambda expression*? No. It's a *parenthesized expression* that contains a *lambda expression*.

The compiler makes a small spec violation here and discards the parenthesis for you. Second:  

int M() { return 1; }  
Func\<int\> F3() { return M; }  
Func\<int\> F4() { return (M); }

F3 is legal. Is F4? No. Section 7.5.3 states that a parenthesized expression may not contain a method group. Again, for your convenience we (accidentally\!) violate the specification and allow the conversion. Third:  

enum E { None }  
E F5() { return 0; }  
E F6() { return (0); }

F5 is legal. Is F6? No. The spec states that there is a conversion from the literal zero to any enumerated type. "(0)" is not the literal zero, it is a parenthesis followed by the literal zero, followed by a parenthesis. We violate the specification here and actually [allow any compile time constant expression equal to zero, and not just literal zero](http://blogs.msdn.com/ericlippert/archive/2006/03/28/the-root-of-all-evil-part-one.aspx).

So in every case, we allow you to get away with it, even though technically doing so is illegal.

