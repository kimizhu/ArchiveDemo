# Every Tree There Is

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/22/2010 7:05:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/19/every-binary-tree-there-is.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/26/every-program-there-is-part-one.aspx).\]

[![BinaryTrees1](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/EveryTreeThereIs_C707/BinaryTrees1_thumb.png "BinaryTrees1")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/EveryTreeThereIs_C707/BinaryTrees1_2.png)Last time we talked about how the number of binary trees with n nodes is C(n), where C(n) is the nth Catalan number. I asked if there were more or fewer trees – not restricted to binary trees – of size n than there are binary trees of size n. If you worked it out, the answer might have surprised you; it is certainly not immediately obvious.[![BinaryTrees2](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/EveryTreeThereIs_C707/BinaryTrees2_thumb.png "BinaryTrees2")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/EveryTreeThereIs_C707/BinaryTrees2_2.png)

First off, a common response I get to this question is immediately "well, since binary trees are a special case of arbitrary trees, there must be more arbitrary trees." Can you see why this argument is wrong? Binary trees have more structure than arbitrary trees\! There are two binary trees with two nodes: one with the child on the right of the root, one with the child on the left of the root. There is only one arbitrary tree with two nodes; there is no difference between the "left" and "right" child.

As it turns out, the answer to my question is that except for the trivial cases of zero- and one-node trees, there are always more binary trees of size n than trees of size n.

If you actually work it out though you find an interesting pattern; the number of arbitrary trees of size n is C(n-1) – that is, there are 14 binary trees of size 4, and 14 arbitrary trees of size 5.  There are 1430 binary trees of size 8 and 1430 arbitrary trees of size 9. Weird\! (Or, another way of looking at it is that the number of binary trees with n "null leaf nodes" is equal to the number of arbitrary trees with n nodes.)

Clearly this cannot be a coincidence; there must be a one-to-one relationship between the set of binary trees of size n and that of arbitrary trees of size n+1. And in fact the mapping is easy to see if you look at it in just the right way. On the left I’ve drawn a mapping from the first five binary trees of size 4 enumerated by my code of last time and the first five corresponding trees of size 5. (Click on the picture for a larger version.) 

To make it more clear exactly what the relationship is between the two trees, check out the diagram on the right. To turn a binary tree into its corresponding arbitrary tree, you rotate it 45 degrees anticlockwise, stick a root node above the whole thing, and then fix up all the horizontal lines to point to the appropriate parent.

Or, from a data structures point of view, you can represent every node in an arbitrary tree as a reference to the first child and a reference to the next sibling. This is exactly the same as a binary tree, it's just that the labels on the nodes are "first child" and "next sibling", not "left" and "right". The only difference between binary trees and arbitrary trees in this system is that the "right" child of the root is always null, since the root has no siblings. Once you see that the difference between a binary tree and an arbitrary tree is just the names of the fields in the data structure, it becomes apparent that there is a strong relationship between them.

So, interestingly enough, the solution to the second part of my challenge is: **we’re already done**. Since we have a device which enumerates all binary trees, and binary trees are 1-1 with arbitrary trees, then **all we need is a device which can change binary trees to arbitrary trees**. Doing so is left as an exercise, but for interest’s sake, here’s a device which takes a binary tree and gives a string representation of the corresponding arbitrary tree, using the compact syntax for an arbitrary tree I described last time.

 

public static string TreeString(Node node)  
{  
     // Get the arbitrary tree string corresponding to  
     // a given binary tree.  
     var sb = new StringBuilder();  
     Action\<Node\> f = null;  
     f = n =\>  
     {  
         sb.Append("{");  
         for (Node child = n.Left; child \!= null; child = child.Right)  
             f(child);  
         sb.Append("}");  
     };  
     f(new Node(node, null));  
     return sb.ToString();  
}

To get all the trees of size 5, we just ask for all the binary trees of size 4 and print them as trees of size 5:

 

foreach (var n in AllBinaryTrees(4))  
    Console.WriteLine(TreeString(n));

and we get

 

{{}{}{}{}}  
{{}{}{{}}}  
{{}{{}}{}}  
{{}{{}{}}}  
{{}{{{}}}}  
{{{}}{}{}}  
{{{}}{{}}}  
{{{}{}}{}}  
{{{{}}}{}}  
{{{}{}{}}}  
{{{}{{}}}}  
{{{{}}{}}}  
{{{{}{}}}}  
{{{{{}}}}}

Notice that if you remove the outer set of braces on each then we have also solved the problem of “print all the legal combinations of four sets of matched braces”\! If you can enumerate all the binary trees then it turns out you can enumerate all the solutions to dozens of different equivalent problems.

**Next time**: what else can we generate all of?

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/19/every-binary-tree-there-is.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/26/every-program-there-is-part-one.aspx).\]

