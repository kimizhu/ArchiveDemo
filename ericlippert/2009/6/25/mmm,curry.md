# Mmm, Curry

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/25/2009 9:51:00 AM

-----

A recent comment asked why Haskell programmers sometimes write C\# lambdas in this style:

 

Func\<int, Func\<int, int\>\> add = x=\>y=\>x+y;

which is then invoked as

 

sum = add(2)(3);

because of course the first invocation returns a function that adds two, which is then invoked with three. Why do that instead of the more straightforward

 

Func\<int, int, int\> add = (x,y)=\>x+y;

and invoke it as

 

sum = add(2,3);

???

I was going to write a short article on that when I remembered that Wes already had done a better job at it than I likely would. [Read this first before you read on.](http://blogs.msdn.com/wesdyer/archive/2007/01/29/currying-and-partial-function-application.aspx)

Welcome back. I would add to that two interesting facts.

First, that the operation of rewriting an n-parameter function as a bunch of single-parameter functions to achieve “partial application” is called “currying” in honor of Haskell Curry, the logician after whom the programming languages Haskell and Curry are named.

And second, that it is a little-known fact that there is an ugly but sometimes useful way to “curry away” the first parameter of an existing function using reflection.

For example, suppose you've got

 

public class C {  
  public static int M(T t, int x) { whatever }  
}

In C\#, to curry away the first parameter you could simply say

 

T t = something;  
Func\<int, int\> d = x=\>C.M(t, x);

Which is of course a syntactic sugar for creating a hidden nested class with a field t and a method that takes an int, blah blah blah, lots of boring code spit out to do that.

If you need to curry away the first parameter of a method and are using some managed language that does not have anonymous functions, or you are writing a compiler but you don’t feel like spitting a whole bunch of helper methods like the C\# compiler does, you can also do this directly with reflection:

 

T t = something;  
MethodInfo methodInfo = typeof(C).GetMethod("M", BindingFlags.Static | BindingFlags.Public);  
Func\<int, int\> d = (Func\<int, int\>) Delegate.CreateDelegate(typeof(Func\<int, int\>), t, methodInfo);

This works in the .NET framework 2.0 release or higher, and, unfortunately, **requires that T be a reference type.**

Notice that this kind of currying is essentially the same currying that happens when you make an extension method into a delegate. If static method C.M is an extension method extending type T then in C\# you can say

Func\<int, int\> d = t.M;

and we essentially curry away the first argument to the extension method when creating the delegate, using tricks similar to the reflection trick. (We pass the managed address of the extension method to a constructor rather than passing the method info to the factory, but, basically its the same thing behind the scenes.) If you try this on an extension of a value type, [you'll get an error](http://stackoverflow.com/questions/1016033/extension-methods-defined-on-value-types-cannot-be-used-to-create-delegates-why). Sree has just posted [an analysis of why this has to be a reference type](http://blogs.msdn.com/sreekarc/archive/2009/06/25/why-can-t-extension-methods-on-value-type-be-curried.aspx).

