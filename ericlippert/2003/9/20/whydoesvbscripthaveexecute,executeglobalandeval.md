# Why does VBScript have Execute, ExecuteGlobal and Eval?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/20/2003 1:18:00 PM

-----

 

JScript has an extremely powerful (and almost always misused, but that's another story) feature: eval takes a string at runtime and treats that string as though it were part of the compile-time text of the program.  I added that feature to VBScript version 5, but I did it with three methods: Execute, ExecuteGlobal and Eval.  Why three, when JScript makes do with one? Let's start by examining in detail what the JScript "eval" function does.

  

 The JScript "eval" function takes a string, treats the string as JScript code, compiles the string, executes the resulting code, and returns the value of the l*ast expression evaluated* while running the code.

 

 

Now, in JScript (and many other C-like languages) there is a fairly weak distinction between a **statement** and an **expression**.  For example, this is perfectly legal JScript:

 

 

function foo()

{

    1;

    2;

    3 + 4;

}

  

This doesn't do much; it doesn't even return a value\! But that's not the compiler's problem.  The behaviour of the line 3 + 4; for instance is to add three to four and discard the result.  

 

Also note that semicolons are optional in JScript (sort of -- I'll post more about that later.)   

 

 

So when you say eval("3 + 4") in JScript you get seven -- the compiler adds the semicolon on, executes the **statement** 3 + 4; and returns the result of the last **expression** computed.

 

 

Now consider how JScript evaluates expressions that reference variables. The implementation of eval is smart enough to follow JScript's rules for inner scopes shadowing outer scopes:

 

 

var x = 20;

function foo()

{

  var x = 10;

  print(eval("x")); // 10 -- eval uses local

}

 

 

This seems reasonable, right?  But what if you evaluate a declaration?

 

 

var x = 20;

function foo()

{

  eval("var x = 10");

}

foo();

print(x); // 20 -- declaration was local to foo.

 

 

Well, maybe it's silly to want to add a declaration using eval.  But hold on, as we already discussed, named functions are basically just variables.  Suppose you wanted to add a *function* dynamically:

 

 

function foo()

{

  eval("function bar(){ return 123; }");

  print(bar());  // 123

}

foo();

print(bar()); // fails

 

 

Why does the latter fail?  Because the eval is done in the scope of the function foo, so function bar is local to foo.  When foo goes away, so does bar. 

 

 

The long and short of it is that if you want to affect the global name space, you have to do it explicitly.  For example:

   

var bar;

function foo()

{

  eval("function barlocal(){ return 123; } bar = barlocal;");

}

foo();

print(bar()); // succeeds, bar is a global function.

 

 

(And of course, bar now refers to a closure. If foo took any arguments, they would be captured by barlocal.)

 

 

Now, suppose you were tasked with implementing eval in VBScript.  A few salient facts might spring to mind:

 

 

1\) VBScript doesn't have first class functions, so this trick with assigning a local function into global scope won't work. (Well, VBScript has a very weak form of first class functions that I'll discuss later.)

 

 

2\) But contrariwise, it would be a real pain in the rear if the VBScript's "eval" couldn't access local variables, and worked ONLY at global scope.

 

 

3\) VBScript does not have this weird property that expressions are statements.  In fact, you can't determine whether you're looking at an expression or a statement lexically thanks to the assignment and equality operators being the same.  Suppose you see this string:X = Y  Does that mean "set variable X to the value of Y" or does that mean "compare X to Y, leave both the same, and return True or False"? Obviously we want to be able to do *both,* but how do we tell the difference?

 

 

There are *three* things that we need to do: evaluate **expressions**, execute **statements** using **local** scope and execute **statements** using **global** scope.  My philosophy is **when you have three things to do, implement three methods**.  Hence, we have Eval, which takes an expression and returns its value, Execute, which takes a group of statements and executes them in local scope, and ExecuteGlobal which executes them in global scope.

 

 

"*I understand why you need to distinguish between* Eval* and *Execute," I hear you say, "*but why have both* Execute* and *ExecuteGlobal*?  Why not just add an optional -IsGlobal- flag to *Execute*?*"

 

 

Good question.  First of all, in my opinion it is bad coding style to implement public methods which have very different behaviour based on the value of a flag.  You do two things, have two methods.

 

 

Second, Boolean flags are a bad idea because sometimes you want to extend the method even more.  VBScript has this problem in the runtime -- a lot of the methods take an argument which is either 0 for "case insensitive" or 1 for "case sensitive" **or a valid LCID**, for "case sensitive in this locale". What a mess\! (Worse, 1 is a valid locale identifier: Arabic-with-neutral-sublanguage.)

 

In the case at hand, I suppose an enumerated type would be superior to a Boolean and extensible to boot, but still, it makes my skin crawl to see one public method that does two things based on a flag where two methods will do.

