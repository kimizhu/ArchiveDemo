# Protected Semantics, Part Five: More on immutability

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/5/2008 12:52:47 PM

-----

I asked a second follow-up question back in Part Two:

> Suppose you wanted to make this hierarchy an immutable collection, where "Add" and "Remove" returned new collections rather than mutating the existing collection. How would you represent the parenting relationship?

The short answer is "I wouldn't". First off, it sounds an awful lot like what is intended to be represented here is some sort of highly mutable system, where objects are being added and removed from containers. It might be more sensible to represent that thing using mutable data structures rather than immutable data structures.

If you did want to represent the thing with immutable data structures, then I would think twice before attempting to represent parenting at all. Here's why: when you have an immutable tree structure without parents, and you want to make a new tree structure that is based on an old one, typically the only stuff you have to change is from the edited node on up to the root node; usually hierarchies are shallow, so this is not very many nodes.

But this logically implies that the root node changes on every edit. If the root node changes, then all of its children have a new parent, and all of their children then have a new parent... and you've just rewritten the entire tree structure, not just a few nodes.

Now, it is often highly convenient to know what the parent of a given node is. To achieve that, what I'd probably do is try to not need to query the parent of any node until I was at a point where I wanted to ask about a lot of parents, and the tree was likely to remain unedited while I was asking. A quick visitor pass that builds up a hash table mapping every node to its parent would be easy enough to write, and then the parent table could be discarded and its memory reclaimed once I was done.

