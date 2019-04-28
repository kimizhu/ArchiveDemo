<div id="page">

# Not as easy as it looks

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/13/2011 7:01:00 AM

-----

<div id="content">

<div class="mine">

My colleague [Kevin](https://twitter.com/Pilchie) works on (among many other things) the refactoring engine in the C\# IDE. He and I were at the end of last year discussing the possible cases for a hypothetical "eliminate variable" refactoring. I thought that it might be of interest to you guys to get a glimpse of the sorts of issues that the IDE team faces on a daily basis.

First off, what is the "eliminate variable" refactoring? It is seemingly straightforward, but as we'll see, appearances can be deceiving. Suppose you have a local variable that is initialized in its declaration, never written to otherwise, and you have only **one** place where that variable is used. (A generalization of this refactoring is to replace every usage; for simplicity, let's consider only the single usage case.) For example:

<span class="code"> </span>

void M()  
{  
    int x = 2 + R();  
    Q();  
    N(x);  
}

You can eliminate the variable by refactoring the code to:

<span class="code"> </span>

void M()  
{  
    Q();  
    N(2 + R());  
}

Now, obviously this changes the meaning of the code. If Q() throws an exception then now R() is never called, whereas before it was always called. If Q() observed a side effect of R() in the first version, it does not after the refactoring. And so on; there are numerous ways in which this code is different. We assume that the user is all right with these changes; that's why they're using the refactoring.

How, at a high level, would you implement this? It seems straightforward:

  - Find the text of the initializer
  - Find the single usage (which the IDE already knows how to do somehow)
  - Textually replace the single usage with the initializer text
  - Erase the declaration and initialization of the variable

That last step might be slightly tricky if you have something like

<span class="code"> </span>

    int y, x = 2 + R();

because we have to make sure to erase the comma and the declaration and initialization of x, but not erase the "<span class="code">int y;</span>". But that is not hard to deal with.

You also have to think a bit about comments. What should happen here?

<span class="code"> </span>

    // x is blah blah blah  
    int x = 2 /\* blahblah \*/ + R(); // Blah\!  
    Q();  
    N(x);

Should that become

<span class="code"> </span>

    // x is blah blah blah  
    /\* blahblah \*/  // Blah\!  
    Q();  
    N(2 + R());

Or should any of the three comments before, inside or after the declaration be inserted where x used to be used? Notice that if the one that comes after is inserted, it's going to have to be changed to an inline comment, or a newline is going to have to be inserted in the inside of the call. Also, a comment refers to a local variable that has been eliminated, which seems bogus.

Assume that we can solve these problems. Are we done? Let's apply our refactoring to a few test cases. In every case, we eliminate "x".

<span class="code"> </span>

    const short s = 123;  
    int x = s;  
    N(x);

becomes

<span class="code"> </span>

    const short s = 123;  
    N(s);

Is that right? Not necessarily. What if N has two overloads, one for int and one for short? Before we called <span class="code">N(int)</span>; now we are calling <span class="code">N(short)</span>\! This seems qualitatively different than merely reordering a possible side effect; now our refactoring is changing overload resolution, which seems wrong. Really we should be generating

<span class="code"> </span>

    const short s = 123;  
    N((int)s);

OK, is that the right refactoring? Let's keep on trying test cases.

<span class="code"> </span>

    int x = 123;  
    N(ref x);

becomes... what? We can't say <span class="code">N(ref 123)</span> or even <span class="code">N(ref (int)123)</span>.

In this case, **the refactoring engine has to fail.** There is no way to eliminate the variable when what you're passing is a reference to the variable itself.

How about this?

<span class="code"> </span>

    int x = R() + S();  
    N(Q() \* x);

becomes

<span class="code"> </span>

    N(Q() \* R() + S());

Argh, we have now introduced a semantic change because of operator precedence. That should be

<span class="code"> </span>

    N(Q() \* (R() + S()));

How about this?

<span class="code"> </span>

    Func\<int, int\> x = z=\>z+1;  
    object n = x;

becomes

<span class="code"> </span>

    object n = z=\>z+1;

Nope; you can't implicitly convert a lambda to anything other than a delegate or expression type. That's got to be

<span class="code"> </span>

    object n = (Func\<int, int\>)(z=\>z+1);

Whew. What have we learned so far? To eliminate <span class="code">T x = expr</span>, there are scenarios where you can use:

  - nothing at all; give an error
  - <span class="code">expr</span>
  - <span class="code">(expr)</span>
  - <span class="code">(T)expr</span>
  - <span class="code">(T)(expr)</span>

An exercise for the reader: **find scenarios in which none of the above are good enough either**. Can you find a scenario in which you have to say <span class="code">((T)expr)</span>?  How about <span class="code">((T)(expr))</span>? The latter was the worst I could find easily; are there any more that I missed?

Understandably, we would not want to implement this refactoring in a hypothetical future version of Visual Studio if every time you did an elimination of a variable, the compiler conservatively inserted four parens plus a cast; what we really need are heuristics which can tell us when each of the above forms is *necessary*. That is a surprisingly tricky problem in language analysis\!

And these are just the issues that Kevin and I came up with in a few minutes of whiteboard doodling; it is entirely possible that on deeper analysis, we'll find more interesting problems.

Next time you use the refactoring tools in Visual Studio, think about everything that is happening behind the scenes to make those tools work seamlessly. Like playing the piano, **the unseen effort required to make it look easy is enormous**.

</div>

</div>

</div>

