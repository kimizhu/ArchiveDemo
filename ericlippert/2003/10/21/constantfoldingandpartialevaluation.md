# Constant Folding and Partial Evaluation

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/21/2003 1:31:00 PM

-----

A reader asks "*is there any reason why VBScript doesn't change* str = str & "1234567890" & "hello"  *to* str = str & "1234567890hello" *since they are both constants?*" 

 

 

Good question.  Yes, there are reasons. 

 

 

The operation you're describing is called **constant folding**, and it is a very common compile-time optimization.  VBScript does an *extremely* limited kind of constant folding.  In VBScript, these two programs generate exactly the same code at the call site:

 

 

const foo = "hello"

print foo

 

 

is exactly the same as 

 

 

print "hello"  
  

That is, the code generated for both says "pass the **literal string** "hello" to the print subroutine".  If foo had been a variable instead of a constant then the code would have been generated to say "pass the **contents** of variable foo..."  But the VBScript code generator is smart enough to realize that foo is a constant, and so it does not generate a by-name or by-index lookup, it just slams the constant right in there so that there is no lookup indirection at all.

 

 

The kind of constant folding you're describing is *compile-time evaluation of expressions which have all operands known at compile time*.  For short, let's call it **partial evaluation**.  In C++ for example, it is legal to say

 

 

const int CallMethod = 0x1;

const int CallProperty = 0x2;

const int CallMethodOrProperty = CallMethod | CallProperty;

 

 

The C++ compiler is smart enough to realize that it can compute the third value itself.  VBScript would produce an error in this situation, as the compiler is not that smart.  Neither VBScript nor JScript will evaluate constant expressions at compile time.

 

 

An even more advanced form of constant folding is to determine which functions are pure functions -- that is, functions which have no side effects, where the output of the function depends solely on the arguments passed in.  For example, in a language that supported pure functions, this would be legal:

 

 

const Real Pi = 3.14159265358979;

const Real Sine60 = sine( Pi / 3);  // Pi / 3 radians = 60 degrees

 

 

The sine function is a pure function -- there's no reason that it could not be called at compile time to assign to this constant.  However, in practice it can be very difficult to identify pure functions, and even if you can, there are issues in calling arbitrary code at compile time -- like, what if the pure function takes an hour to run?  That's a long compile\!  What if it throws exceptions?  There are many practical problems.

 

 

The JScript .NET compiler does support partial evaluation, but not pure functions.  The JScript .NET compiler architecture is quite interesting.  The source code is lexed into a stream of tokens, and then the tokens are parsed to form a parse tree.  Each node in the parse tree is represented by an object (written in C\#) which implements three methods: Evaluate, PartialEvaluate and TranslateToIL.  When you call PartialEvaluate on the root of the parse tree, it recursively descends through the tree looking for nodes representing operations where all the sub-nodes are known at compile time.  Those nodes are evaluated and collapsed into simpler nodes.  Once the tree has been evaluated as much as is possible at compile time, we then call TranslateToIL, which starts another recursive descent that emits the IL into the generated assembly.

 

 

The Evaluate method is there to implement the eval function.  JScript Classic (which everyone thinks is an "interpreted" language) *always* compiles the script to bytecode and then interprets the bytecode -- even eval calls the bytecode compiler in JScript Classic.  But in JScript Classic, a bytecode block is a block of memory entirely under control of the JScript engine, which can release it when the code is no longer callable.  In JScript .NET, we compile to IL which is then jitted into machine code.  If JScript .NET's implementation of eval emitted IL, then that jitted code would stay in memory until the appdomain went away\!  This means that a tight loop with an eval in it is essentially a **memory leak** in JScript .NET, but not in JScript Classic.  Therefore, JScript .NET actually implements a true interpreter\!  In JScript .NET, eval generates a parse tree and does a full recursive evaluation on it.   

 

 

I'm digressing slightly.  You wanted to know why the script engines don't implement partial evaluation.  Well, first of all, implementing partial evaluation would have made the script engines considerably more complicated for very little performance gain.  And if the author does want this gain, then the author can easily fold the constants "by hand".   

 

 

But more important, partial evaluation makes the process of compiling the script into bytecode much, much longer as you need to do yet another complete recursive pass over the parse tree.  That's great, isn't it?  I mean, that's trading increased compilation time for decreased run time.  What could be wrong with that?  Well, it depends who you ask.

 

 

From the ASP implementers' perspective, that would indeed be great.  An ASP page, as I've already discussed, only gets compiled once, on the first page hit, but might be run many times.  Who cares if the **first** page hit takes a few milliseconds longer to do the compilation, if the subsequent million page hits each run a few microseconds faster?  And so what if this makes the VBScript DLL larger?  ASP updates are distributed to people with fast internet connections.

 

 

But from the IE implementers' perspective, partial evaluation is a step in the wrong direction.  ASP wants the compilation to go slow and the run to go fast because they are generating the code once, calling it a lot, and generating strings that must be served up as fast as possible.  IE wants the compilation to be as fast as possible because they want as little delay as possible between the HTML arriving over the network and the page rendering correctly.  They're never going to run the script again after its generated once, so there is no amortization of compilation cost.  And IE typically uses scripts to run user interface elements, not to build up huge strings as fast as possible.  Every microsecond does NOT count in most UI scenarios -- as long as the UI events are processed just slightly faster than we incredibly slow humans can notice the lag, everyone is happy.

