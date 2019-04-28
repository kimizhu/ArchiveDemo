# Maybe there's something wrong with the universe, but probably not

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/25/2011 1:30:15 PM

-----

No kidding, I was just walking down a hallway in my building when I overhead the following quite loud conversational fragment through an open doorway:

**Angry woman's voice**: "Why are you in the ladies room?\! You are the *third* man to... oh no."

Like Hobbes, it's the moment of dawning comprehension that I live for - the exact moment when she realized that she, not everyone else, was in the wrong room was readily apparent. (One wonders what the first two gentlemen did, since clearly they did not successfully disabuse the lady of her error.) Since the building across the courtyard from mine has a mirror-imaged layout, this is a very easy mistake to make if you are visiting from the other building.

I contrast that moment of dawning comprehension with Dr. Crusher's similar moment in that memorable 1990 episode of Star Trek: The Next Generation when she realizes that [she's not crazy, it's the entire universe that is wrong](https://www.youtube.com/watch?v=_nSlND2g7vc&#t=179s). When faced with an absurd and unexpected situation - the gradual disappearance of first the crew and then the entire universe - she at least considers that [she's the crazy one](http://memory-alpha.org/wiki/Remember_Me#ACT_Two).

Unlike most people, I encounter compiler and library bugs all day long in my job, though mostly ones that I caused in the first place. (Sorry\!) But even still, when I am writing "normal" code (rather than test cases designed to break the compiler or regress previous bugs), I try to ensure that my attitude upon encountering an unexpected situation is that **I'm the crazy one**. Usually it's my code that is wrong, or my misunderstanding the output, rather than a compiler or library bug.

As the authors of "[The Pragmatic Programmer](http://www.pragprog.com/the-pragmatic-programmer)" point out in their third chapter, "**select() isn't broken**" - if you are writing perfectly normal code then odds are good you are *not* the first person to discover what should be an obvious problem in a well-tested product. If you think you've found a bug in the math library, maybe you have. Or maybe you've actually passed radians to a method that takes degrees, or forgotten to take floating point rounding error into account, or some other such thing. The more obvious the problem, the more likely it is that you're the crazy one. If the code doesn't compile and you think it should, it could be a bug in the compiler. But read the error message carefully; it is probably telling you what is wrong with the code.

If you think you've found a C\# compiler bug, please, by all means bring it to our attention; post it on [Connect](http://connect.microsoft.com/), or have the community take a gander at it via [StackOverflow](http://www.stackoverflow.com/) or one of the [Microsoft forums](http://social.msdn.microsoft.com/Forums/en-US/csharplanguage/threads), and if you want, [send me a link](http://blogs.msdn.com/b/ericlippert/contact.aspx) to the problem. **(Please don't use the "contact" link to send me source code directly; the hostile-email sterilization code that filters that text is very aggressive about stripping out things that look potentially harmful. It makes code almost illegible.)** There certainly are bugs in the compiler and the more we get good information on, the better. Including a small-but-complete program that reproduces the problem and the version number of the compiler you're using is a big help. But first, do stop and take a good hard look at the code and think about whether it is more likely to be a problem with the code or a problem with the compiler. Don't be one of those people who sends me angry, profane emails about a problem that you caused yourself; that's just embarrassing.

