# The void is invariant

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/29/2009 10:30:00 AM

-----

\[UPDATES below\] 

A while back I described [a kind of variance that we’ve supported since C\# 2.0.](http://blogs.msdn.com/ericlippert/archive/2007/10/19/covariance-and-contravariance-in-c-part-three-member-group-conversion-variance.aspx) When assigning a method group to a delegate type, such that both the selected method and the delegate target agree that their return type is a reference type, then the conversion is allowed to be covariant. That is, you can say:

 

Giraffe GetGiraffe() { … }  
…  
Func\<Animal\> f = GetGiraffe;

This works logically because anyone who calls f must be able to handle any animal that comes back. The actual method claims to only return animals, and in fact, makes the stronger claim to only return giraffes.

This works out in the CLR because the bits that make up a reference to an instance of Giraffe are exactly the same bits that make up a reference to that Giraffe interpreted as an instance of Animal. We can allow this magical conversion to happen because the CLR guarantees that it will all just work out without going in there and having to futz around with the bits.

This is why this trick only works with reference types. A method that returns, say, a double cannot be converted via a covariant conversion to a delegate type that expects the method to return an object. Somewhere there would have to be code emitted that takes the returned double and boxes it to object; the bits of a double and the bits of a reference to an object boxing a double are completely different.

But why doesn’t this trick work with void types? Here we have a method that returns some sort of success or failure code. Maybe we don’t care what it returns.

 

static bool DoSomething(bool b)  
{  
  if (b) return DoTheThing();  
  else return DoTheOtherThing();  
}

Action\<bool\> action = DoSomething;

This doesn’t work. Why not? The caller of the action is not even going to *use* the returned value, so it doesn’t matter one bit what it is\! Shouldn’t “void” be considered *a supertype of all possible types* for the purposes of covariant return type conversions from method groups to delegate types?

No, and I’ll tell you why.

Consider what happens when you do this:

 

bool x = DoSomething(true);

We spit out IL that does the following:

(1) put true on the IL stack – the stack gets one deeper  
(2) call DoSomething – the argument is removed from the stack and the return value is placed on the stack.  Net, the stack stays the same size as before  
(3) stuff whatever on top of the stack into local variable x – the stack now returns to its original depth.

Now consider what happens when you do this:

 

DoSomething(true);

We spit out IL that does the first two steps as before. But we cannot stop there\! There is now a bool on the IL stack which needs to be removed. We generate a pop instruction to represent the fact that the returned bool has been discarded.

Now consider what happens when you do this:

 

action(true);

The compiler believes that action is a void-returning method, so it does not generate a pop instruction. If we allowed you to stuff DoSomething into the action, then we would be allowing you to misalign the IL stack\!

But didn’t I say “[the stack is an implementation detail?](http://blogs.msdn.com/ericlippert/archive/2009/04/27/the-stack-is-an-implementation-detail.aspx)” Yes, but *that’s a different stack*. The CLI specification describes a “virtual machine” which passes around arguments and returned values on a stack. An implementation of the CLI is required to make something that behaves like the specified machine, but it is not required to do so in any particular manner. It is not required to use the million-bytes-per-thread stack supplied to each thread by the operating system as its implementation of the IL stack; that’s a convenient structure to use, of course, but it’s an implementation detail that it does so.

(As an aside: when we implemented the script engines, we also first specified our own private stack-based virtual machine. When we implemented it, we decided to put the information about “return addresses” – that is, “what code do I run next?” on the system stack, but we put arguments and return values of script functions in a stack-shaped block of memory that we allocated on our own. This made building the JScript garbage collector easier.)

In practice, the jitter uses the system stack for some things and registers for other things. Return values are actually often sent back in a register, not on the stack. But that implementation detail doesn’t help us out when deciding what the conversion rules are; we have to assume that the implementation can do no more than what the CLI specification says. Had the CLI specification said “the returned value of any function is passed back in a ‘virtual register’” rather than having it pushed onto the stack, then we could have made void-returning delegates compatible with functions that returned anything. You can always just ignore the value in the register. But that’s not what the CLI specified, so that’s not what we can do.

\[UPDATE\]

A number of people have asked in the comments why we do not simply generate a helper method that does what you want. That is, when you say

 

Action\<bool\> action = DoSomething;

realize that as

 

static void DoSomethingHelper(bool b)  
{  
   bool result = DoSomething(b); // result is ignored  
}  
...  
Action\<bool\> action = DoSomethingHelper;

We could do that. But where would you like the line to be drawn? Should you be able to assign a reference to a method that returns an int to a Func\<Nullable\<int\>\>? We could spit a helper method that converts the int to a nullable int. What about Func\<double\>? We could spit a helper method that converts the int to a double. What about Func\<object\>? We could spit a helper method that boxes the int, unexpectedly allocating memory off the heap every time you call it. What about a Func\<Foo\> where there is a user-defined implicit conversion from int to Foo?

We could be spitting arbitrarily complex fixer-upper methods that would seamlessly "do what you meant to say", and we have to stop somewhere. The exact semantics of what we do and do not fix up would have to be designed, specified, implemented, tested, documented, shipped to customers and maintained forever. Those are costs. Plus, [every time we add a new conversion rule to the language we add breaking changes](http://blogs.msdn.com/ericlippert/archive/2007/08/31/future-breaking-changes-part-two.aspx). The costs of those breaking changes to our customers have to be factored in.

But more fundamentally, one of the design principles of C\# is "if **you say something wrong then *we tell you* rather than trying *to guess what you meant***". JScript is deliberately a "muddle on through and do the best you can" language; C\# is not. If what you want to do is make a delegate to a helper method then you express that intention by going right ahead and making that method.

