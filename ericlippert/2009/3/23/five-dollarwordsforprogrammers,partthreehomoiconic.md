# Five-Dollar Words For Programmers, Part Three: Homoiconic

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/23/2009 4:33:42 PM

-----

Jeff Atwood was kind enough to once more [give me the shout-out in his blog the other day](http://www.codinghorror.com/blog/archives/001244.html). Thanks Jeff\!

This inspires me to continue my series on five-dollar words for programmers. Here’s one that I only learned relatively recently, when I helped write the code that translates a lambda expression into an expression tree which represents the content of the lambda: **[homoiconic](http://en.wikipedia.org/wiki/Homoiconic)**.

[![Expression Trees](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/FiveDollarWordsForProgrammersPartThreeHo_BE81/Expression%20Trees_3.png "Expression Trees")](http://msdn.microsoft.com/en-us/library/bb397951.aspx) A language is said to be **homoiconic** if the **representation** of the program can be seen as a **data structure** expressible in that language. With expression lambdas being convertible to expression trees (which can then be compiled into code at runtime), C\# 3.0 is *somewhat* homoiconic. But it pales in comparison to, say, LISP, where pretty much everything you can do in the language you can also represent as structurally isomophic data.

Something I personally would like to see more of in C\# in the future is greater homoiconicity. We could extend expression trees to statement trees, declaration trees, program trees, and so on. This series of steps would enable increasingly powerful and interesting **metaprogramming scenarios**.

C\# suffers from the lack of a metalanguage. We absolutely do not want to go towards the horrible and primitive metaprogramming language exemplified by the C preprocessor language. We already have a great language, C\#, so why not use C\# as a metalanguage? Wouldn’t it be nice to **make C\# its own metalanguage**? And once we do that, then we get into some truly strange loops, where we could use those data structures in the compiler implementation itself\!

This is a long way off and might never happen, but a guy can dream.

