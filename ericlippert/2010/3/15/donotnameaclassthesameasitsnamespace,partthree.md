# Do not name a class the same as its namespace, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/15/2010 6:53:00 AM

-----

(This is part three of a four part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/11/do-not-name-a-class-the-same-as-its-namespace-part-two.aspx), part four is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/18/do-not-name-a-class-the-same-as-its-namespace-part-four.aspx).)

**Part Three: Bad hierarchical design**

The reason we humans invented hierarchies in the first place is to organize a complicated body of stuff such that there’s a well-defined place for everything. Any time you see a hierarchy where there are two levels with the same name, something is messed up in the design of that hierarchy. And any time you see a hierarchy where one of the interior nodes has a single child, again, something is probably messed up.

Krzysztof points out in the annotated Framework Design Guidelines that the fundamental point of namespaces is not actually to allow you to disambiguate two things with the same name. (Ideally there would never be a situation where two things had the same name in the first place; coming up with a mechanism to enable that problem and then deal with it seems counterproductive.) Rather, **the point of namespaces is to organize types into a hierarchy that is easy to understand**.

That is, the point of namespaces is not just to keep similarly-named things separated, but rather, to group things that have something in common together so that you can find them. If you don’t think that there are two or more things that could go into a namespace, then it is probably not a good namespace.

My original example was a namespace MyContainers.List containing a class List. Could any other class go into MyContainers.List? No. The right design is to either move the class List into the MyContainers namespace, or to make the namespace MyContainers.Lists, plural, and have it contain more than one thing, say, MutableList and ImmutableList.

The commonality that groups a set of types into a namespace could be anything. System.Collections.Generic groups collection types by an implementation detail: they’re all generic. System.IO groups types by related functionality. Top-level namespaces like “System” and “Microsoft” group things by whether they are part of the core functionality of the platform, or are Microsoft-specific extensions to it. But the point is that each of these namespaces groups a large number of things by some shared characteristic. A namespace containing a type of the same name indicates a failure in the design of the hierarchy.

Next time: It makes a bad situation worse.

(This is part three of a four part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/11/do-not-name-a-class-the-same-as-its-namespace-part-two.aspx), part four is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/03/18/do-not-name-a-class-the-same-as-its-namespace-part-four.aspx).)

