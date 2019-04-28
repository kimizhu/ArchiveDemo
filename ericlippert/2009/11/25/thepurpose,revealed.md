# The Purpose, Revealed

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/25/2009 9:03:00 AM

-----

Interestingly enough, no one correctly guessed why I needed code in the compiler to transform a method argument list into a form that batched up the side effecting expressions, stuck their values in variables, and then called the method with the side-effect-free variable references. There certainly were some fascinating ideas in the comments though\!

The reason is quite mundane. We had a bug in the code that performs semantic analysis of calls with named arguments. One of our rules in C\# that I've talked about a lot on this blog is that sub-expression evaluation occurs in left-to-right order. If you have a method void M(int x, int y) {} and then a call site M(y : F(), x : G()); we cannot simply generate the code as though you'd written M(G(), F()). The side effects of F have to happen before the side effects of G, so we generate code as though you'd said temp1 = F(); temp2 = G(); M(y : temp1, x : temp2); and now we can generate the call as M(temp2, temp1); without worrying about re-ordering a side effect.

The bug was that the rewriter was not getting the reordering correct if the parameters were ref/out or the arguments were locals without side effects. As I often do when fixing a bug, I reasoned that the initial bug had been caused because the rewriter was under-specified.

The fix will get into the initial release of C\# 4, but the problems that Pavel identified with it -- that certain obscure side effects such as the order in which bad array accesses throw exceptions, or the order in which static class constructors execute, are not correctly preserved by my algorithm, will not be fixed for the initial release. Doing these correctly requires us to be able to generate temporaries of "reference to argument" types, which we've never done before and which our IL generator was not built to handle. Had we discovered the bug much earlier in the cycle, we probably could have applied a more solid fix, but it is too late now.

Thanks again to Pavel for the great analysis, and thanks to our old friend nikov for reporting the bug in the first place.

And happy thanksgiving to my readers in the United States\! As I do every year, I'm roasting a turkey to feed fourteen people. I highly recommend the "brine the bird then roast it upside-down" method described in The Joy Of Cooking. It's worked perfectly for me for the last ten years straight.

