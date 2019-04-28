# Cast operators do not obey the distributive law

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/27/2010 6:55:00 AM

-----

Another interesting question from [StackOverflow](http://stackoverflow.com/questions/1171717/c-conditional-operator). Consider the following unfortunate situation:

 

object result;  
bool isDecimal = GetAmount(out result);  
decimal amount = (decimal)(isDecimal ? result : 0);

The developer who wrote this code was quite surprised to discover that it compiles and then throws “invalid cast exception” if the alternative branch is taken.

Anyone see why?

In regular algebra, multiplication is “distributive” over addition. That is q \* (r + s) is the same as q \* r + q \* s. The developer here was probably expecting that casting was distributive over the conditional operator. It is not. This is not the same as

 

decimal amount = isDecimal ? (decimal)result : (decimal)0;

which is in fact the correct code here. Or, better still:  
  
decimal amount = isDecimal ? (decimal)result : 0.0m;

The problem faced by the compiler is that the **type of the conditional expression must be consistent for both branches**; the language rules do not allow you to return object on one branch and int on the other.

We choose the best type based on the types we have in the expression itself, not on the basis of types that are outside the expression, like the cast. Therefore the choices are object and int. Every int is convertible to object but not every object is convertible to int, so the compiler chooses object. Therefore this is the same as

 

decimal amount = (decimal)(isDecimal ? result : (object)0);

And therefore the zero returned is a **boxed int**. The cast then unboxes the boxed int to decimal. As we’ve already discussed at length, [it is illegal to unbox a boxed int to decimal](http://blogs.msdn.com/ericlippert/archive/2009/03/19/representation-and-identity.aspx). That throws an invalid cast exception, and there you go.

