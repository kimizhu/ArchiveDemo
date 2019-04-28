# Always write a spec, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/19/2009 6:45:00 AM

-----

Joel had a great series of articles many years ago about [the benefits of writing functional specifications](http://www.joelonsoftware.com/articles/fog0000000036.html), that is, specifications of how the product looks to its users. I want to talk a bit about technical specifications, that is, a specification of how something actually works behind the scenes. A while back, I described how I wrote a seemingly-trivial piece of code by [first writing the technical spec, and then writing the code to exactly implement the spec](http://blogs.msdn.com/ericlippert/archive/2009/06/01/bug-psychology.aspx), and the test cases to test each statement of the spec. I use this technique frequently; it almost always saves me time and effort. If you don't write a spec first, it's very easy to get bogged down in a morass of bugs, and hard to know when you're actually done writing the code.

Here's an example of another seemingly-trivial function that [smart people gave a good half-dozen completely wrong answers to because they didn't write a spec](http://stackoverflow.com/questions/921180/c-round-up/926806#926806). Notice how once I state what the spec is, the code follows very naturally.

I thought I might go through a recent example of a slightly less trivial problem. I needed a function in the compiler which could take a valid list of arguments A1 to an applicable method M, and give me back two lists. The first result list is a list of *local variable declarations*, L, the second, another argument list, A2. Here's the kicker: evaluating each member of A2 *must have no side effects*. And, the combination of executing L followed by M(A2) must give *exactly the same results* as executing M(A1). (I also knew that the members of A1 had already had all the conversions generated to make them match the types of M's parameter list, and so on. Overload resolution was completely done at this point.)

Why I needed such a function is not particularly important; perhaps you can speculate as to why I needed it.

So I started writing a spec, starting with the description above: the function T takes a valid list... blah blah blah.

That's not enough of a spec to actually write code.

I realized that some expressions in A1 might have side effects, and some might not. If an expression had side effects, then I could declare a temp local, execute the side effects, assign the result to the local, and then use the local as the expression in the same place in A2.

Super. Now I can write more sentences in my spec.

-----

For each argument x in A1, we potentially generate a temporary value as follows: First, if x has no side effects, then simply keep x in the appropriate place in A2. Otherwise, if x does have side effects, then generate a temporary variable declaration in L, var temp = x. Then put an expression that evaluates the temporary variable in the appropriate place in A2.

-----

Then I thought "can I correctly implement this spec?"

I started thinking about the possible cases for the input, A1. The members of A1 could be passed by value, out, or ref; I already knew they were correct, since the particular method M was an applicable method of argument list A1. In this particular point in the compilation process, there was no need to worry about whether there were "params" arguments, whether there were missing arguments with default values, and so on; that had already been taken care of. I also noted that whether they were "ref" or "out" was irrelevant; ref and out are the same thing behind the scenes. As far as the compiler is concerned, the only difference between them is that the definite assignment rules are different for each. So we had two cases for each argument: it's being passed by value, or by ref/out.

If it's being passed by ref/out, then what we actually do is we pass the managed address of a variable to the method. In M(ref int x), the type of x is actually a special type int&, "managed reference to variable of type int", which is not a legal type in C\#. That's why we hide it from you by requiring a goofy "ref" syntax any time you want to talk about a managed reference to a variable.

Unfortunately, our IL generation pass was written with the assumption that all local variables are never of these magic managed-reference-to-variable types. I was then faced with a choice. Either come up with a technique for working with this limitation, or rewrite the most complicated code in the IL gen to support this scenario. (The code which figures out how to deal with managed references is quite complex.) I decided to opt for the former strategy. Which meant more spec work, since our spec now doesn't handle these cases\!

-----

For each argument x in A1, we potentially generate a temporary value as follows:

First, if x has no side effects, then simply keep x in the appropriate place in A2.

Otherwise, if x does have side effects **and corresponds to a value parameter**, then generate ... etc.

Otherwise, if x does have side effects and corresponds to a ref/out parameter, **then a miracle happens**.

-----

Clearly that last bit needed some work.

So I wrote down all the possible cases I could think of. Clearly x has to be a variable if its type is "reference to variable", so that makes for a small number of cases:

-----

Otherwise, if x does have side effects and corresponds to a ref/out parameter, then the possible cases are:

\* x is a local variable  
\* x is a value parameter  
\* x is an out parameter  
\* x is a ref parameter  
\* x is a field of an instance  
\* x is a static field  
\* x is an array element  
\* x is a pointer dereferenced with \*  
\* x is a pointer dereferenced with \[ \]

and for each, a miracle happens.

-----

Again, clearly this is not good enough. I re-examined my list to see if I could eliminate any of these cases. Locals, parameters and static fields never have side effects, so we can eliminate them.

Also, at this point in our analysis, I knew that pointer dereferences of the form "pointer\[index\]" had already been rewritten into the form "\*(pointer + index)", which meant that in practice, we would never actually hit that last case in this algorithm; the second-last case would take care of both.

-----

Otherwise, if x does have side effects and corresponds to a ref/out parameter, then the possible cases are:

\* x is a field of an instance  
\* x is an array element  
\* x is a pointer dereferenced with \*  
  
and for each, a miracle happens.

-----

I then started to think of what side effects could happen for each. We could have "ref instance.field", "ref array\[index\]", or "ref \*pointer", and "instance", "array", "index" and "pointer" can all be expressions that have a side effect. ("field" cannot, it merely names a field.) So now we can use the same specification as before:

-----

Otherwise, if x does have side effects and corresponds to a ref/out parameter, then the possible cases are:

  - x is ref/out instance.field: in this case, add var temp=instance to L and ref/out temp.field to A2.
  - x is ref/out array\[index\]: in this case, add var t1 = array and var t2 = index to L and ref/out t1\[t2\] to A2.
  - x is ref/out \*pointer: in this case, add var temp = pointer to L and ref/out \*temp to A2.

-----

And now we have something I can implement. So I sent this proposed spec around for review, while I started plugging away at the implementation.

**This spec is wrong.** Can you spot the bugs in my implementation that my coworkers found by reading my spec?

Next time, the thrilling conclusion.

