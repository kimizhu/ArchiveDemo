# Functions are not frames

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/31/2003 12:54:00 PM

-----

I just realized that on my list of features missing from JScript.NET "fast mode" I forgot about the caller property of functions.  In compatibility mode you can say

 

 

function foo(){bar();}

function bar(){print(bar.caller);}

foo();

 

 

In fast mode this prints null, in compatibility mode it prints function foo(){bar();}. 

 

 

Eliminating this feature does make it possible to generate faster code -- keeping track of the caller of every function at all times adds a fair amount of complexity to the code generation.  But just as importantly, this feature is simply incredibly broken by its very design. The problem is that **the function object is completely the wrong object to put the** caller** property upon in the first place**.  For example:

 

 

function foo(x){bar(x-1);}

function bar(x)

{

  if (x \> 0)

    foo(x-1);

  else

  {

    print(bar.caller.toString().substring(9,12));

    print(bar.caller.caller.toString().substring(9,12));

    print(bar.caller.caller.caller.toString().substring(9,12));

    print(bar.caller.caller.caller.caller.toString().substring(9,12));

  }

}

function bla(){foo(3)}

blah();

 

 

This silly example is pretty straightforward -- the global scope calls bla.  bla calls foo(3), calls bar(2), calls foo(1), calls bar(0), prints out the call stack.  So the call stack at this point should be foo, bar, foo, bla, right?  So why does this print out foo, bar, foo, bar?

 

 

Because the caller property is a property of the function object and it returns a function object.  bar.caller and bar.caller.caller.caller **are the same object**, so of course they have the same caller property\! Clearly this is completely broken for recursive functions.  What about multi-threaded programs, where there may be multiple callers on multiple threads?  Do you make the caller property different on different threads?   

 

 

These problems apply to the arguments property as well.  Essentially the problem is that the notion we want to manipulate is **activation frame**, not **function object**, but function object is what we've got.  To implement this feature properly you need to access the stack of activation frames, where an activation frame consists of a function object, an array of arguments, and a caller, where the caller is another activation frame.  Now the problem goes away -- **each activation frame in a recursive, multi-threaded program is unique**.  To gain access to the frame we'd need to add something like the *this* keyword -- perhaps a *frame* keyword that would give you the activation frame at the top of the stack.

 

 

That's how *I* would have designed this feature, but in the real world we're stuck with being backwards compatible with the original Netscape design.  Fortunately, the .NET reflection code lets you walk stack frames yourself if you need to.  Though it doesn't integrate perfectly smoothly with the JScript .NET notion of functions as objects, at least it manipulates frames reasonably well.

