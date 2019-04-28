# Standard Generic Delegate Types

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/21/2006 2:29:00 PM

-----

Hey all, I'm back from my vacation. Two weeks of reading, sailing, kayaking and visiting with old friends has left me a lot more relaxed and sunburnt than when I left. I could use another week, but it's also good to be back.

We're introducing a lot of new features in C\# 3.0 which, when combined to form LINQ are really interesting and powerful, but, like the component parts of Voltron, are pretty interesting and powerful just on their own. Lambda expressions, for example, are not just useful for making query comprehensions work. They make functional-style programming in C\# 3.0 much more elegant than the somewhat clunky anonymous method syntax from C\# 2.0.

We're also introducing a new standard generic delegate type in the LINQ libraries to make delegate declaration easier. In the old days, to create a function that takes an int and returns a function from int to int, you'd have to do something like this:

delegate int D1(int y);  
delegate D1 D2(int x);  
//...  
D2 makeAdder = delegate(int x){  
  return delegate(int y){  
    return x + y;  
  };  
};  
D1 addTen = makeAdder(10);  
Console.WriteLine(addTen(20));  

In the new world we'll have these definitions in a standard library:

delegate R Func\<R\>();  
delegate R Func\<A1, R\>(A1 a1);  
delegate R Func\<A1, A2, R\>(A1 a1, A2 a2);  
delegate R Func\<A1, A2, A3, R\>(A1 a1, A2 a2, A3 a3);  
//...etc, up to some reasonable number of arguments  

so that you can use them plus lambdas to make the code above far more concise:

Func\<int, Func\<int, int\>\> makeAdder = x=\>y=\>x+y;  
Func\<int, int\> addTen = makeAdder(10);  
Console.WriteLine(addTen(20));  

Here's an interesting fact: there are delegate types which can be defined using the old-fashioned syntax but *cannot* be defined using the newfangled generic syntax. Void delegates, generic delegates and delegates with out or ref parameters are obvious examples. Can you think of any other examples? Next time on FAIC, I'll post an interesting one.

