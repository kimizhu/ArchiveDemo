# Every Binary Tree There Is

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2010 6:34:00 AM

-----

\[This is the first part of a series on generating every string in a language. The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/22/every-tree-there-is.aspx).\]

The other day I wrote a little algorithm that did some operation on binary trees. I wanted to test it. I whipped up a few little test cases and it seemed fine, but I wasn’t quite satisfied. I was pretty confident, but maybe there was some odd binary tree topology that I hadn’t considered which would cause a bug. I reasoned that there have got to be only a *finite* number of binary tree topologies of a given size. **I’ll just try all of them.**

Before I go on, I need a compact notation for a binary tree. Here’s my tree node:

 

class Node  
{  
    public Node Left { get; private set; }  
    public Node Right { get; private set; }  
    public Node(Node left, Node right)  
    {  
        this.Left = left;  
        this.Right = right;  
    }  
}

Pretty straightforward –- left node, right node, and that’s it. Notice that for the sake of clarity of this article I’ve removed the data that would normally be stored in the tree node. Let’s assume that they’re numbers for now. The interesting bit here is how I represent the tree as an extremely compact string. A null tree node is represented as x. A nonempty tree node in my “valueless” tree is represented as (left right). Consider this tree:  

  1  
/ \\  
x   2  
   / \\  
  x   x

The 2 node has two null children, so it is  (xx). The 1 node has one null child on the left and the 2 node on the right, so it is (x(xx)). Make sense? We can write a bit of recursive code that produces such a string:  

public static string BinaryTreeString(Node node)  
{  
    var sb = new StringBuilder();  
    Action\<Node\> f = null;  
    f = n =\>  
    {  
        if (n == null)  
            sb.Append("x");  
        else  
        {  
            sb.Append("(");  
            f(n.Left);  
            f(n.Right);  
            sb.Append(")");  
        }  
    };  
    f(node);  
    return sb.ToString();  
}

How can we enumerate all possible binary trees of a given size? I reasoned recursively: There is one tree with zero non-null nodes in it: x. That’s our base case. Now, pick a number. Four. We want to enumerate all the trees with four non-null nodes in them. Suppose we already know how to enumerate all the trees with three, two, one and zero nodes in them. Let's call the set of binary trees with n nodes in it B(n). Suppose we make all possible combinations of B(x) and B(y) such that a member of B(x) is to the left of the root and a member of B(y) is to the right. I'll write that B(x)B(y). In this notation the trees with four non-null nodes in them have to be in one of these four possible sets: B(0)B(3), B(1)B(2), B(2)B(1), or B(3)B(0). Obviously this generalizes; we can enumerate all the trees with k nodes in them by going through each of k cases, where each case requires us to solve the problem on some tree sizes smaller than k. Perfect for a recursive solution. Here’s the code.  

static IEnumerable\<Node\> AllBinaryTrees(int size)  
{  
    if (size == 0)  
        return new Node\[\] { null };  
    return from i in Enumerable.Range(0, size)  
           from left in AllBinaryTrees(i)  
           from right in AllBinaryTrees(size - 1 - i)  
           select new Node(left, right);  
}

Once more, **LINQ makes an algorithm read much more like its specification than the equivalent program with lots of nested for loops.** And sure enough, if we run  

foreach (var t in AllBinaryTrees(4))  
    Console.WriteLine(BinaryTreeString(t));

we get all fourteen trees with four non-null nodes:

 

(x(x(x(xx))))  
(x(x((xx)x)))  
(x((xx)(xx)))  
(x((x(xx))x))  
(x(((xx)x)x))  
((xx)(x(xx)))  
((xx)((xx)x))  
((x(xx))(xx))  
(((xx)x)(xx))  
((x(x(xx)))x)  
((x((xx)x))x)  
(((xx)(xx))x)  
(((x(xx))x)x)  
((((xx)x)x)x)

And now I have a device which generates tree topologies that I can use to test my algorithm.

How many such trees are there, anyway? Seems like there might be rather a lot of them.

The number of binary trees with n nodes is given by the [Catalan numbers](http://en.wikipedia.org/wiki/Catalan_number), which have many interesting properties. The nth Catalan number is determined by the formula (2n)\! / (n+1)\!n\!, which grows exponentially. (See Wikipedia for several proofs that this is a closed form for the Catalan numbers.) The number of binary trees of a given size is:

<table>
<tbody>
<tr class="odd">
<td>0</td>
<td>1</td>
</tr>
<tr class="even">
<td>1</td>
<td>1</td>
</tr>
<tr class="odd">
<td>2</td>
<td>2</td>
</tr>
<tr class="even">
<td>4</td>
<td>14</td>
</tr>
<tr class="odd">
<td>8</td>
<td>1430</td>
</tr>
<tr class="even">
<td>12</td>
<td>208012</td>
</tr>
<tr class="odd">
<td>16</td>
<td>35357670</td>
</tr>
</tbody>
</table>

So perhaps my plan to try all binary trees of a particular size is not a great one; by the time you get into the mid-teens there are way too many to reasonably try all of them in a short amount of time. Still, I think this is a handy device to have around.

A challenge for you: Suppose we forget about binary trees for a moment and consider **arbitrary trees**. An arbitrary tree node can have zero, one, or any finite number of child tree nodes. Let's say that a non-empty arbitrary tree is notated as a list of child trees in braces. So {{}{}{{}}} is the tree

 

    1  
   /|\\  
  2 3 4  
      |  
      5

Because the 2, 3 and 5 nodes each have no children, they are notated as {}. Make sense? Note that order matters; {{}{}{{}}} and {{}{{}}{}} are different trees even though they have a similar structure.

My challenge questions are (1) are there more or fewer arbitrary trees with n nodes than there are binary trees with n nodes? and (2) can you come up with a mechanism for enumerating all the arbitrary trees?

\[This is the first part of a series on generating every string in a language. The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/04/22/every-tree-there-is.aspx).\]

