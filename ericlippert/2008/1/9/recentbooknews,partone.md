# Recent Book News, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/9/2008 12:04:22 PM

-----

Good day and Happy New Year everyone, I hope you had a restful and joyous festive holiday season. I sure did.

Before I get back into my series on immutable data structures I have a few news items regarding books.

[![C\# 3.0 Design Patterns by Judith Bishop](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/RecentBookNewsPartOne_7F8F/Bishop_thumb.jpg)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/RecentBookNewsPartOne_7F8F/Bishop_2.jpg)

First off, I am pleased to announce the availability of [C\# 3.0 Design Patterns by Judith Bishop](http://patterns.cs.up.ac.za/). An amusing story about this book:

I was lucky enough to be one of the technical reviewers relatively early on in the genesis of the book. (I do technical editing as a hobby; it's nice to have a hobby that brings in pocket money instead of spending it\!) As often happens when the technical reviewers are done reading the book, I was asked if I would contribute a endorsing quote for the back of the book. Absolutely, I replied, it's a fine book, I would be happy to. I whipped up "*C\# 3.0 Design Patterns brings the frequently abstruse world of design patterns into sharp focus with pragmatic C\# 3.0 implementations*" in about 30 seconds and sent it off to the publisher.

First thing the next morning -- which, apropos of nothing, was my birthday -- the senior editor drops me a line to say that quote looks great, oh, by the way, how about you write us a foreword where that's the last sentence, we go to press, uh, today, but we could hold the presses until tomorrow if we need to.

Yikes\!

Twenty minutes later, this is what I came up with. If it seems a little rushed, that's because it was. The foreword is the least important page of the book, including the colophon, so I'm not particularly worried.

Congratulations to Judith Bishop on her latest accomplishment in publishing, and many thanks to everyone at O'Reilly who made the editing process very pleasant indeed.

-----

When you’re faced with a problem to solve (and frankly, who isn’t these days?) the basic strategy usually taken by we computer people is called “divide and conquer”. It goes like this:

  - Conceptualize the specific problem as a set of smaller sub-problems.
  - Solve each smaller problem.
  - Combine the results into a solution of the specific problem.

Reducing complex problems down to the level of twiddling the states of a few billion bits is what we do all day. But “divide and conquer” is not the *only* possible strategy. We can also take a more generalist approach:

  - Conceptualize the specific problem as a special case of a more general problem.
  - Somehow solve the general problem.
  - Adapt the solution of the general problem to the specific problem.

Design patterns are among the major tools in the toolboxes of those who espouse the generalist approach. If you look at samples from a broad spectrum of software solutions, you will find that though the specifics may vary widely, there is often an underlying structural similarity. (Searching a file system for a file with a particular attribute is in some sense structurally similar to searching an annotated parse tree for a symbol with a particular type.) *Design patterns codify general solutions to common problems.*

The ultimate example of the generalist approach is of course the design and implementation of programming languages themselves. As problem solving tools go, it is hard to get more general than a programming language like C\#. When designing new programming languages (or new versions of old programming languages) we think about common problems that are faced every day by real developers and figure out how to create a language which solves them in a general, aesthetically pleasing and powerful way that is broadly applicable.

We want to embed the most useful and powerful abstractions so deeply into the language infrastructure that you barely even consciously register them as being there anymore. Patterns like “local variable” or “procedure call” or “while loop” are so much a part of the air we all breathe that we don’t even think of them as patterns anymore.

Furthermore, we want to make a language in which patterns which are useful but perhaps not quite so fundamental are nevertheless relatively straightforward to implement clearly and elegantly. A class in C\# may be marked as “static”, or “abstract”, or “sealed”, but not as “singleton”. That was a deliberate choice of the language designers. However, implementing a singleton class in C\# is still relatively easy.

The gray zone in between “clearly foundational” and “occasionally useful” is where the interesting design challenges lie. Our observations of design patterns used by real-world developers in C\# (and other languages) strongly drive the design process for new versions.

Consider for example how you would implement an iterator pattern on a linked list in C\# 1.0. You would end up defining an enumerator class to represent a position in the list containing a lot of boring boilerplate code (which impedes readability), and the solution would not be very reusable. The notion of “enumerate a set of things” is sufficiently applicable to a wide variety of problems that it met the bar for inclusion as a first class language concept. In C\# 2.0 with its “yield return” statement the compiler can generate all the boring code for you, and the generic type system makes iterating over a set of things typesafe no matter what the “things” are.

All of this is a long way to say just why it is that I am so very excited about Language Integrated Query (LINQ) in C\# 3.0. We believed that iterating over collections of things was a great start, but that we could do so much more. *Sorting*, *filtering*, *grouping*, *joining*, *projecting* and *transforming* data are also fundamental operations which are useful in pretty much every domain. Whether you are writing a ray tracer, a compiler, an XML reader or an online banking security system, odds are good that you are going to need to manipulate collections of *something* in a rich way.

By moving these concepts out of domain-specific object models and into a general-purpose programming language hopefully we solve those more general problems. We additionally hope though that by adding C\# 3.0’s query expressions, lambda expressions, extension methods, initializer expressions, expression trees, and so on to the already rich set of C\# 2.0 and 1.0 features, we make it easier to elegantly implement all sorts of other useful design patterns.

And that is also why I am excited about this book; C\# 3.0 Design Patterns brings the frequently abstruse world of design patterns into sharp focus with pragmatic C\# 3.0 implementations. I look forward to seeing where developers can go with these tools and this language, and what useful patterns we can build into the infrastructures of future languages.

Eric Lippert  
Seattle, Washington  
November 30, 2007

