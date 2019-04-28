# Immutability in C\# Part Seven: More on Binary Trees

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/19/2007 3:01:00 PM

-----

Lots of good comments on my previous post. To briefly follow up:

  - One of the downsides of immutable tree implementations is that usually the tree must be built from the leaves up, which is not always convenient. We'll look at implementations which hide this fact from the user in future posts.

<!-- end list -->

  - One smartypants pointed out that sure, this tree can have cycles -- if you have *another* binary tree implementation that has cycles and then paste such a cyclic tree in. True; what I meant was that *using only the stated implementation*, it's impossible to create a tree with cycles. If you are hell bent on making a bad tree, you can surely do that. Again, in a future post we'll have a tree which is built up by calling tree "mutating" operations, and those really will be guaranteed acyclic even in a world where there are other bad implementations hanging around.

And finally, one reader correctly identified the problem with my recursive implementation of in-order iteration. A tree of maximum height h ends up allocating O(h) iterator objects. The recursive calls get O(h) deep. If h is very large, this could blow the call stack. And if the tree has n nodes, each with an average height of O(h), then iterating each node will require O(h) recursive calls apiece. Therefore the total time cost in calls for iterating the entire tree is O(n h). 

In a binary tree with n nodes, h is always between log n and n, so that means that this iterator has best case asymptotic performance of O(n log n) and worst case  of O(n<sup>2</sup>) in time, and uses between O(log n) and O(n) in both call stack and heap space. That's pretty lousy.

A better implementation would be to write a tree traversal algorithm which does not use recursion. [Use an explicit stack rather than the call stack:](http://blogs.msdn.com/ericlippert/archive/2005/08/01/recursion-part-two-unrolling-a-recursive-function-with-an-explicit-stack.aspx)

public static IEnumerable\<T\> InOrder\<T\>(this IBinaryTree\<T\> tree)  
{  
    IStack\<IBinaryTree\<T\>\> stack = Stack\<IBinaryTree\<T\>\>.Empty;  
    for (IBinaryTree\<T\> current = tree; \!current.IsEmpty || \!stack.IsEmpty; current = current.Right)  
    {  
        while (\!current.IsEmpty)  
        {  
            stack = stack.Push(current);  
            current = current.Left;  
        }  
        current = stack.Peek();  
        stack = stack.Pop();  
        yield return current.Value;  
    }  
}

This consumes O(n) time, O(h) heap space, and O(1) call stack space. It is also **painful** to read and analyze compared to the slow naive implementation. I would very much like to add new syntax to a hypothetical future version of C\# which would be a syntactic sugar for the code above. If, hypothetically, we were prioritizing features for unannounced future versions of C\#, unfortunately that one would not be our highest priority, so I wouldn't expect it any time soon. [My colleague Wes has written a great article that goes into more details about the problem above](http://blogs.msdn.com/wesdyer/archive/2007/03/23/all-about-iterators.aspx) and ways we might solve it in the future, so check that out if you want the details.

I have the code for the next few blog posts written, but odds are good that I'm not going to have time to write the surrounding text until after the festive holiday season is over. I hope you have a wonderful time during the remainder of 2007 and we'll see you bright and early in 2008 for more fabulous adventures\!

