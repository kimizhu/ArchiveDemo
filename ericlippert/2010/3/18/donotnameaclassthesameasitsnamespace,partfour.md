# Do not name a class the same as its namespace, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/18/2010 6:56:00 AM

-----

(This is part four of a four part series; part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/15/do-not-name-a-class-the-same-as-its-namespace-part-three.aspx).)

**Part Four: Making the problem worse**

I said earlier that the fundamental reason for namespaces in the first place was organization of types into a hierarchy, not separation of two things with similar names. But suppose you *are* putting something into a namespace because you have two things that are of the same name and need to be kept separate. Suppose you reason “I’m going to put List into its own namespace because List could conflict with another class named List. The user needs to be able to qualify that.”

OK, that’s fine; put List into MyContainers then. But *why* would you then repeat the process and put List into a child namespace in MyContainers? **The most plausible reason is that the level of disambiguation achieved so far is insufficient; some other entity named List is going to be in scope in a code region where elements of MyContainers are also in scope.**

Let us posit that as our cause for creating a new namespace MyContainers, and then creating a new sub-namespace, MyContainers.X, to be the declaration space of List. What name should we choose for X? If the whole point is that something else named List is in scope somewhere that elements of MyContainers are in scope, then choosing “List” for “X” is making the problem worse, not better\! Before you had two things named List in scope. Now you have *three* things named List, two of which are in scope. This is making it more confusing without solving the problem at hand: that there are two things named List in scope.

Any one of these four reasons is enough to avoid this bad practice; avoid, avoid, avoid.

(This is part four of a four part series; part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/15/do-not-name-a-class-the-same-as-its-namespace-part-three.aspx).)

