# Recursion, Part One: Recursive Data Structures and Functions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/27/2005 6:00:00 AM

-----

The first thing to wrap your head around is recursively defined data structures. Let's start with something simple.  Think about the abstract idea of “list”.  Most people think of a “list” as an ordered collection of “items”, one after the other, with a beginning and an end.  That’s a very non-recursive way to think about a list, but it’s the usual way of doing so.

The recursive way to think about a list emphasizes the similarity of a fragment of the list to the whole list.  We might recursively define a list as either

  - the empty list with zero items on it, or
  - an item followed by a list.

This certainly appears to be a circular definition, but it’s really not. It’s more sort of a “spiral” definition. You start off with something extremely simple; it’s harder to get more simple than a list with no elements. Then you provide a rule for turning something simple into something *very slightly* more complicated. The recursive part of the definition means that you can take a list of any size, starting with zero, and construct a list one bigger than it, by sticking an item on to the beginning. It’s not a truly circular definition because there is always a “base case”. There is always some point from which you begin, and upon which you can build. But the true power of the recursive definition comes from going the other way: if you have something that meets the definition, then there is usually a way to move back towards the base case, getting simpler and simpler as you go. That’s what recursive programming is all about. All recursive functions follow the same basic pattern:

  - Am I in the base case? If so, return the easy solution.
  - Otherwise, figure out some way to make the problem closer to the base case and solve that problem.

That’s why you can get away with calling a function from itself – because you know that the recursion will eventually bottom out when you’ve found the base case. Now, for list processing, a recursive solution is usually overkill. The iterative solution of “run down the list in a loop” is usually good enough. But many other problems are naturally solved by other recursively defined data structures. For example, consider the highly useful binary tree. A binary tree is either:

  - an empty tree, or
  - a value associated with two binary search trees, called the left and right child trees

For example, this is a binary tree:           E  
         / \\  
        /   \\  
       B     G  
      / \\     \\  
     A   C     H  The children of A, C and H, and the left child of G, are empty trees. (Sharp eyed readers will have noted that this particular example is a binary *search* tree, because every parent is between its children. We might discuss properties of search trees later.) Pretty picture, but how are we going to represent this thing in code?  Suppose you’ve got a binary tree: in J**Script var mytree =  
{  
  left:  
  {  
    left:  
    {  
      left: null,  
      value : "A",  
      right: null  
    },  
    value: "B",  
    right:  
    {  
      left: null,  
      value: "C",  
      right: null  
    }  
  },  
  value:"E",  
  right :  
  {  
    left: null,  
    value: "G",  
    right:  
    {  
      left: null,  
      value: "H",  
      right: null  
    }  
  }  
}; Now we can use this recursive definition to write recursive functions that answer questions about this tree.  For the rest of this series I'll use as my motivating example one of the simplest questions you can ask about a binary tree: **what is the deepest you can go in a given tree?**  We have a recursive definition, so come up with a recursive solution. What’s the base case?  An empty tree.  Clearly the deepest you can go in an empty tree is zero steps.  What’s the recursive case? If you haven’t got an empty tree then you’ve got a value with two children.  Each child is itself a tree. The deepest you can go from this point is one (counting yourself), plus the depth of the deepest child. And we've got a function that gives you the depth of a tree, so let's call it on each child: function treeDepth(curtree) {  
  if (curtree == null)  
    return 0;  
  return 1 + Math.max(treeDepth(curtree.left), treeDepth(curtree.right));  
} It's magically delicious.  It's like having n dominoes lined up.  If you knock over the last one, and you know that every domino knocked over also knocks over the previous, then you know that they'll all go down eventually. Formally, we could use complete induction to show that this works for all trees.  We know that it works for trees of height zero. As an inductive hypothesis, assume that it works for trees of height \<= k.  Since a tree of height k+1 has one subtree of height k and another of height \<=k, by hypothesis the recursion correctly calculates their heights and computes the max as k+1, and we're done. (Of course, how do we know that complete induction works? That question takes us into Peano's Axioms and other topics in the foundations of mathematical logic. Perhaps another time...) Next time: Trouble in paradise: the out-of-stack error rears its ugly head.

