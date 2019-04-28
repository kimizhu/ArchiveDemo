# Eval is Evil, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/1/2003 1:03:00 PM

-----

 

The eval method -- which takes a string containing JScript code, compiles it and runs it -- is probably the most powerful and most misused method in JScript. There are a few scenarios in which eval is invaluable.  For example, when you are building up complex mathematical expressions based on user input, or when you are serializing object state to a string so that it can be stored or transmitted, and reconstituted later.

 

 

However, these worthy scenarios make up a tiny percentage of the actual usage of eval.  In the majority of cases, eval is used like a sledgehammer swatting a fly -- it gets the job done, but with too much power.  It's slow, it's unwieldy, and tends to magnify the damage when you make a mistake.  Please spread the word far and wide: **if you are considering using** eval** then there is probably a better way. ** Think hard before you use eval.

 

 

Let me give you an example of a typical usage.   

 

 

\<span id="myspan1"\>\</span\>

\<span id="myspan2"\>\</span\>

\<span id="myspan3"\>\</span\>

\<script language="jscript"\>

function setspan(num, text)

{

    eval("myspan" + num + ".innerText = '" + text + "'");  
}

\</script\>

 

 

Somehow the program is getting its hands on a number, and it wants to map that to a particular span.  What's wrong with this picture?

 

 

Well, pretty much everything.  **This is a horrid way to implement these simple semantics.**  First off, what if the text contains an apostrophe?  Then we'll generate

 

 

myspan1.innerText = 'it ain't what you do, it's the way thacha do it';  
  

Which isn't legal JScript.  Similarly, what if it contains stuff interpretable as escape sequences?  OK, let's fix that up.

 

 

eval("myspan" + num).innerText = text;  
  

**If you have to use **eval**, **eval** as little of the expression as possible, and only do it once.  **I've seen code like this in real live web sites:

 

 

if (eval(foo) \!= null && eval(foo).blah == 123)

    eval(foo).baz = "hello";

 

 

Yikes\!  That calls the compiler three times to compile up the same code\!  People, eval** starts a compiler**.  Before you use it, ask yourself whether there is a better way to solve this problem than starting up a compiler\!   

 

 

Anyway, our modified solution is much better but still awful.  What if num is out of range?  What if it isn't even a number?  We could put in checks, but why bother?  We need to take a step back here and ask what problem we are trying to solve.   

 

 

We have a number.  We would like to map that number onto an object.  How would you solve this problem if you didn't have eval?  This is not a difficult programming problem\!  **Obviously an array is a far better solution:**

 

 

var spans = new Array(null, myspan1, myspan2, myspan3);

function setspan(num, text)

{

  if (spans\[num\] \!= null)

    spans\[num\].innertext = text;

}

 

 

And since JScript has string-indexed associative arrays, this generalizes to far more than just numeric scenarios.  **Build any map you want.  JScript even provides a convenient syntax for maps\!**

 

 

var spans = { 1 : mySpan1, 2 : mySpan2, 12 : mySpan12 };

 

 

Let's compare these two solutions on a number of axes:

 

 

**Debugability**: what is easier to debug, a program that dynamically generates new code at runtime, or a program with a static body of code?  What is easier to debug, a program that uses arrays as arrays, or a program that every time it needs to map a number to an object it compiles up a small new program?

 

 

**Maintainability**: What's easier to maintain, a table or a program that dynamically spits new code?

 

 

**Speed**: which do you think is faster, a program that dereferences an array, or a program that starts a compiler?

 

 

**Memory**: which uses more memory, a program that dereferences an array, or a program that starts a compiler and compiles a new chunk of code every time you need to access an array?

 

 

There is absolutely no reason to use eval to solve problems like mapping strings or numbers onto objects.  Doing so dramatically lowers the quality of the code on pretty much every imaginable axis.   

 

 

It gets even worse when you use eval on the server, but that's another post.

