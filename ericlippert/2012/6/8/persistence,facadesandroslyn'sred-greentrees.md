# Persistence, Facades and Roslyn's Red-Green Trees

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/8/2012 12:00:50 PM

-----

We decided early in the Roslyn design process that the primary data structure that developers would use when analyzing code via Roslyn is the **syntax tree**. And thus one of the hardest parts of the early Roslyn design was figuring out how we were going to implement syntax tree nodes, and what information they would proffer up to the user. We would like to have a data structure that has the following characteristics:

  - Immutable.
  - The form of a tree.
  - Cheap access to parent nodes from child nodes.
  - Possible to map from a node in the tree to a character offset in the text.
  - **Persistent**.

By *persistence* I mean the ability to *reuse most of the existing nodes in the tree* when an edit is made to the text buffer. Since the nodes are immutable, there's no barrier to reusing them, [as I've discussed many times on this blog](http://blogs.msdn.com/b/ericlippert/archive/tags/immutability/). We need this for performance; we cannot be re-parsing huge wodges of text every time you hit a key. We need to re-lex and re-parse only the portions of the tree that were affected by the edit (\*), because we are potentially re-doing this analysis between every keystroke. When you try to put all five of those things into one data structure you immediately run into problems:

  - How do you build a tree node in the first place? The parent and the child both refer to each other, and are immutable, so which one gets built first?
  - Supposing you manage to solve that problem: how do you make it persistent? You cannot re-use a child node in a different parent because that would involve telling the child that it has a new parent. But the child is immutable.
  - Supposing you manage to solve that problem: when you insert a new character into the edit buffer, the absolute position of *every node that is mapped to a position after that point* changes. This makes it very difficult to make a persistent data structure, because any edit can change the spans of most of the nodes\!

But on the Roslyn team we routinely do impossible things. We actually do the impossible by keeping *two* parse trees. The "green" tree is immutable, persistent, has no parent references, is built "bottom-up", and every node tracks its *width* but not its *absolute position*. When an edit happens we rebuild only the portions of the green tree that were affected by the edit, which is typically about O(log n) of the total parse nodes in the tree. The "red" tree is an immutable *facade* that is built around the green tree; it is built "top-down" *on demand* and thrown away on every edit. It computes parent references by *manufacturing them on demand as you descend through the tree from the top*. It manufactures absolute positions by computing them from the widths, again, as you descend. You, the consumer of the Roslyn API, only ever see the red tree; the green tree is an implementation detail. (And if you use the debugger to peer into the internal state of a parse node you'll in fact see that there is a reference to *another* parse node in there of a different type; that's the green tree node.) Incidentally, these are called "red/green trees" because those were the whiteboard marker colours we used to draw the data structure in the design meeting. There's no other meaning to the colours.

The benefit of this strategy is that we get all those great things: immutability, persistence, parent references, and so on. The cost is that this system is complex and can consume a lot of memory if the "red" facades get large. We are at present doing experiments to see if we can reduce some of the costs without losing the benefits.

-----

(\*) Determining what those portions of the tree are is quite tricky; I might blog about that at a later date. An edit that, for example, adds "async" to a method can cause the parse of "await(foo);" in the method body to change from an invocation to a usage of the await contextual keyword.

