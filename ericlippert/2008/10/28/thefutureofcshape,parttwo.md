# The Future of C\#, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/28/2008 11:01:00 AM

-----

Well, I *intended* to spend the last three weeks blogging about C\# design process in anticipation of our announcements at the PDC, and then I got crazy busy at work and never managed to do so\!

As I'm sure you know by now, we have announced the existence and feature set of that hitherto hypothetical language C\# 4.0. Of course I'll be blogging extensively about that over the next year; today though I want to pick up where I left off last time and talk about the tragedy of Object Happiness Disease.

OHD is a disorder believed to be related to [Thread Happiness Disease](http://blogs.msdn.com/ericlippert/archive/2004/02/15/the-tragedy-of-thread-happiness-disease.aspx), the belief that threads are awesome and that you should therefore make as many as you possibly can. OHD is an even more common condition which afflicts developers at all levels of experience. Symptoms of OHD in C\# developers include:

  - making utterances like "I don't like extension methods because they're not object-oriented"
  - requesting that multiple class inheritance be added to the language
  - writing interfaces only intended to be implemented by one class
  - designing every class to enable derivation, whether it needs it or not
  - and so on.

In short, the Object Happy believe that object-oriented programming is a good in of itself and that OOP principles are objectively (ha ha) good principles.

I don't share these beliefs.

Now, don't get me wrong. I believe that OOP is frequently useful. In fact, it is frequently the best available tool for a great many real-world jobs. OOP works particularly well when you have:

  - multi-person teams of software enginneers,
  - working on solving practical problems,
  - by logically associating the problem data with the functions performed upon them,
  - in a type hierarchy that sensibly models the "is a" and "can do" relationships between the various parts.

And *that* is where its goodness comes from: OOP techniques make real people more productive in solving real problems and thereby creating value. **The techniques are not good in of themselves, they're good because they're practical.**

Perhaps surprisingly, our goal in making C\# is *not* to make an object-oriented language. Our goal is **to make a compelling and practical language for general-purpose application development on our platforms, and thereby enable our customers to be successful.** Making an object-oriented language called C\# is just a means to that end, not an end in of itself.

So, yes, the oft-heard criticism that "*extension methods are not object-oriented*" is entirely *correct*, but also rather *irrelevant*. Extension methods certainly are not object-oriented. They put the code that manipulates the data far away from the code that declares the data, they cannot break encapsulation and talk to the private state of the objects they appear to be methods on, they do not play well with inheritance, and so on. They're procedural programming in a convenient object-oriented dress.

They're also *incredibly convenient* and *make LINQ possible*, which is why we added them. The fact that they do not conform to some philosophical ideal of what makes an object-oriented language was not really much of a factor in that decision.

Here's another way to think about it.

I had a discussion with Jim Miller a while back about the meaning of the term "functional language" in the context of Scheme. To paraphrase somewhat, Jim's position was along the lines that a "functional language" is one in which the design of the language makes it **easy to program in a functional style** and **difficult (or impossible\!) to program in a non-functional style**. For example, Scheme does allow mutation of shared state -- a very non-functional-style feature -- but the convention in Scheme is to call attention to the fact that you are doing so by marking mutating functions with a "\!". It is *possible* to program in an object-oriented style in Scheme, but the language resists you at every turn.

I certainly see the merits of Jim's position, but I take a weaker stance. When I say "functional language", I mean a language in which it is **possible without undue difficulty to program in a functional style**. I do not require the language to *work against you* if you try to program in a non-functional style in it. For example, I would call JScript a functional language. It is certainly possible to program in a non-functional style in JScript, but you can also treat JScript as Scheme with a slightly goofy syntax if you really want to.

I think about "object-oriented language" the same way; C\# is an object-oriented language not because every feature of the language strictly adheres to OO philosophy. Rather, because as a practical matter it is *possible* to program in an OO style in C\# if that's what you want. Adding features that enable other styles of programming does not make C\# less of an OO language.

Next time, I'll talk a bit about the "theme" of C\# 4.0 and how that influenced the design process. (And if you absolutely positively cannot wait to hear about some of the new language feature in detail, try my colleague [Chris Burrows' blog](http://blogs.msdn.com/cburrows/archive/2008/10/27/c-dynamic.aspx).)

