# Write-only variables considered harmful? Or beneficial?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/23/2007 10:00:00 AM

-----

I haven't forgotten that I promised to describe one more place where we insert an explicit conversion automatically. But before that, a quick digression.

Reader [Phil Haack](http://haacked.com/) has written [some good tips for writing clear code](http://haacked.com/archive/2007/04/20/write-readable-code-by-making-its-intentions-clear.aspx) and quotes [an old post of mine](http://blogs.msdn.com/ericlippert/archive/2004/06/14/155316.aspx). However I thought that I would talk a bit about why it is sometimes a good idea to violate his last tip, which is basically "eschew write-only locals". His example is:

 

public void SomeMethod2()  
{  
  bool x = DoSomething();  
  //some other code.  
  //...  
  //end of method. x is never used  
}  

Normally C\# warns on all variables and fields which are never read, never written, etc. But in this case we suppress the warning on purpose if the assignment is not a constant expression. 

This is because there is no good way in the Visual Studio debugger to say "show me the return value of the last function call". Though I would agree were you to sensibly point out that the way to solve this is to *fix the debugger*, given that I have no ability to fix it, we need a solution in C\# for our customers.

Therefore we allow this assignment without producing a warning, and you can then very easily step through the debugger and see what functions returned what values when by looking at the locals window. If you build with optimizations turned on then of course the local is never generated in the IL.

This calls out a factor in code writing which is often glossed over. We ought to all know by now that the most important audience to consider when writing production code is not the compiler, but rather future maintenance programmers who will have to understand it. But we often forget that they don't come to understand it by simply *reading* the code; most of the time that I have to learn a piece of unfamiliar code in a big hurry I am looking at it *live, running in a debugger*. If the code works well with the debugger then it will be much easier to grasp.

As with all design problems, there are tradeoffs. Say we're building up an object which represents a mathematical expression. Which is better?

 

var add =  
  Add(  
    Multiply(  
      Constant(2),  
      Constant(3)),  
    Constant(4));  

or

 

var c2 = Constant(2);  
var c3 = Constant(3);  
var mult = Multiply(c2, c3);  
var c4 = Constant(4);  
var add = Add(mult, c4);

Both have essentially the same number of lines of code. The optimizing compiler will generate exactly the same IL for both. But the former lexically emphasizes the structure of the resulting object, whereas the latter lexically emphasizes the order in which things actually get done. Moreover, the latter is much easier to debug if you want to know what the result of each stage along the way is.

Historically I have tended to favour the first pattern, but I am coming around to the second more and more these days in my own code. I find the former more elegant but harder to debug, and I spend a lot of time debugging. So Phil, don't go off on people who do this in C\# too hard; they may be making your life easier when you have to debug something in the future.

