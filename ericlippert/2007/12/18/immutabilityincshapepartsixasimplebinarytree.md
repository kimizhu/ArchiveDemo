# Immutability in C\# Part Six: A Simple Binary Tree

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/18/2007 2:30:00 PM

-----

OK, we've gotten pretty good at this by now. A straightforward implementation of an immutable generic binary tree requires little comment on the basics. A binary tree is either empty, or a value, left tree and right tree:

 

public interface IBinaryTree\<V\>  
{  
    bool IsEmpty { get; }  
    V Value { get; }  
    IBinaryTree\<V\> Left { get; }  
    IBinaryTree\<V\> Right { get; }  
}

And our implementation has, as always, a singleton empty tree. One minor point is that unlike previous data structures, where the structure itself knew how to build more of the same, this one does not. There's no obvious "insert" operation in a binary tree, so we'll just leave it up to the implementer. (A binary tree is essentially defined by its *shape*, not by the operations one can perform on it.) In this case, we'll just make a constructor:

 

public sealed class BinaryTree\<V\> : IBinaryTree\<V\>  
{  
    private sealed class EmptyBinaryTree : IBinaryTree\<V\>  
    {  
        public bool IsEmpty { get { return true; } }  
        public IBinaryTree\<V\> Left { get { throw new Exception("Empty tree"); } }  
        public IBinaryTree\<V\> Right { get { throw new Exception("Empty tree"); } }  
        public V Value { get { throw new Exception("Empty tree"); } }  
    }  
    private static readonly EmptyBinaryTree empty = new EmptyBinaryTree();  
    public static IBinaryTree\<V\> Empty { get { return empty; } }  
  
    private readonly V value;  
    private readonly IBinaryTree\<V\> left;  
    private readonly IBinaryTree\<V\> right;  
  
    public bool IsEmpty { get { return false; } }  
    public V Value { get { return value; } }  
    public IBinaryTree\<V\> Left { get { return left; } }  
    public IBinaryTree\<V\> Right { get { return right; } }  
    public BinaryTree(V value, IBinaryTree\<V\> left, IBinaryTree\<V\> right)  
    {  
        this.value = value;  
        this.left = left ?? Empty;  
        this.right = right ?? Empty;  
    }  
}

Note that another nice feature of immutable data structures is that **it** **is impossible to accidentally (or deliberately\!) create a tree which contains a cycle**. In a mutable tree you could do something stupid like making the root a child of one of the leaves (or, for that matter, a child of itself\!) Then you have to either live with the possibility that a user will create a cyclic "tree" and hope for the best (or, in other words, crash and/or hang) or you have to build yourself a cycle detector. Cycle detectors have potentially serious performance implications. It may be best to simply prevent the situation from arising in the first place.

A feature conspicuous by its absence from the above implementation is enumeration. Unlike a stack or a queue, there's no obvious best order in which to enumerate the members a binary tree. First off, there's the question of whether to enumerate in depth-first or breadth-first order, and then the matter of whether the parent comes before, after or between the children.

We could create extension methods which did various different enumerations on a binary tree. Here is a **bad implementation** of an "in order" traversal (depth first, parent comes between the children -- so-called because in a binary search tree, this enumerates the nodes in sorted order.)

 

public static IEnumerable\<V\> InOrder\<V\>(this IBinaryTree\<V\> tree)  
{  
    // BAD IMPLEMENTATION, DO NOT DO THIS  
    if (tree.IsEmpty) yield break;  
    foreach(V v in tree.Left.InOrder()) yield return v;  
    yield return tree.Value;  
    foreach(V v in tree.Right.InOrder()) yield return v;  
}

This *seems* perfectly straightforward and sensible, but this implementation is actually quite flawed. We can do a lot better. What is wrong with this implementation of enumeration, and how would you fix it? (Hint: start your analysis with the assumption that the tree has maximum height h and total number of nodes n.)

Next time: a better implementation of traversal, and then after that, some more complex kinds of immutable binary tree.

