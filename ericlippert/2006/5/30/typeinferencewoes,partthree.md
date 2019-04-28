# Type inference woes, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/30/2006 9:58:00 AM

-----

There were a lot of good comments and questions posted in the last two entries which deserve answers. However, I am once more off to the [beaver-shark infested shores of the great Canadian inland sea](http://blogs.msdn.com/ericlippert/archive/2004/07/12/181265.aspx) for my annual summer vacation. I'll get to them when I'm back, in late June.

Before I go I should mention what algorithms we're considering to solve the "infer the best type for this set of expressions" problem that is going to crop up all over the show in C\# 3.0.

We've got four candidates right now. Suppose you've got a set of expressions, E. The set of their types is T.

Algorithm One: pick the unique type in T to which every type in T is implicitly convertible.

This has the advantage of being simple and already in the specification. It has the disadvantages of being incompatible with existing behaviour and not handling typeless expressions well.

Algorithm Two: pick the unique type in T to which every expression in E is implicitly convertible.

This has the advantage of being what we actually implement. However it has the disadvantages of turning some corner cases that should work according to the spec (such as literal-zero-vs-short) into error cases.

We could combine the two algorithms. Consider the subset of T consisting of all types in T to which every expression in E is implicitly convertible. Call this subset S.

Algorithm three: pick the unique type in S to which every type in S is implicitly convertible.

This algorithm has considerable charm. As far as I've been able to determine, it is 100% backwards compatible with existing non-error behaviour, while at the same time turning those cases which ought to be successful according to Algorithm One back into success cases which agree with the spec. Literal-zero-vs-short would go to int. However, there is a down side, which is addressed by Algorithm Four.

Algorithm Four: pick the unique type in S **from which** every type in S is implicitly convertible. That is, pick the *smallest* type in S, not the largest.

This algorithm also maintains backwards compatibility but would make literal-zero-vs-short go to short. And isn't that better? If you have an implicitly typed array of ten thousand shorts, and you throw a single literal zero in there, you probably want it to stay as an array of shorts. Given the choice of several types to which every expression goes, surely picking the smallest is the best choice.

Algorithm Four is currently the front-runner in our thinking but of course I am blogging about this in the first place because there is still time for feedback. Any thoughts on these choices, or ideas for better algorithms, would be appreciated.

Many thanks to my colleauges [Wes](http://blogs.msdn.com/wesdyer/) and [Peter](http://blogs.msdn.com/peterhal/) for their analyses of these algorithms.

See you when I'm back\!

