# Immutability in C\# Part Eight: Even More On Binary Trees

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/18/2008 2:02:13 PM

-----

Last year we declared a relatively simple interface to represent an immutable binary tree. We noticed that it was different from every other interface that we've declared so far, in that it really said nothing at all about the immutability of the tree. One normally thinks of immutable data types not in terms of their *shape*, but rather in terms of what *operations* may be performed on the data type. Operations are usually either to query the object somehow, or to "modify" it by producing new modified versions of the old immutable object.

This binary tree interface is also unsatisfying because it does not address a *problem* to be solved (other than "organize this data into a binary tree", I suppose.) It seems to be a solution in search of a problem. So let's find a problem and build a better binary tree to solve that problem.

The problem I want to consider today is the implementation of immutable "dictionaries" or "maps". That is, objects which allow you to store a list of "keys" and their associated "values", and then rapidly look up the value associated with the key. Here's a reasonable definition for an immutable dictionary interface:

 

public interface IMap\<K, V\> where K : IComparable\<K\>  
{  
    bool Contains(K key);  
    V Lookup(K key);  
    IMap\<K, V\> Add(K key, V value);  
    IMap\<K, V\> Remove(K key);  
    IEnumerable\<K\> Keys { get; }  
    IEnumerable\<V\> Values { get; }  
    IEnumerable\<KeyValuePair\<K,V\>\> Pairs { get; }  
}

Now, we could easily implement this interface by, say, inventing a class which holds on to an immutable stack of pairs. When you want to add a pair, just push it on to the stack. To find a pair, just enumerate the stack, comparing each key as you go. To remove an item, pop every item off the old stack and onto a new stack, skipping the item to remove.

That would fulfill the contract, but the obvious problem with this approach is that adding is cheap, but removal and searching become O(n) for a dictionary containing n items. A stack-based dictionary with ten thousand items takes between 1 and 10000 comparisons to find an item (and always takes the full 10000 if the item is not found.)

You could optimize this somewhat by keeping the stack sorted... except, oh, that then makes insertion O(n) as well, and lookups really only get faster in the case where you are looking up something that isn't there that happens to come early in the list. When you get to something bigger than it, you can fail then. But lookups for things at the end of the list are still potentially slow.

We can do a lot better than that if we organize the pairs into a binary tree sorted so that every keys in the left subtree is not greater than every key in the right subtree. Such a tree is (for obvious reasons) called a **binary search tree.** We can define a new contract for it which says that this has the shape of a binary tree of values, but also may be treated as a map:

 

public interface IBinarySearchTree\<K, V\> :  
    IBinaryTree\<V\>,  
    IMap\<K, V\>  
    where K : IComparable\<K\>  
{  
    K Key { get; }  
    new IBinarySearchTree\<K, V\> Left { get; }  
    new IBinarySearchTree\<K, V\> Right { get; }  
    new IBinarySearchTree\<K, V\> Add(K key, V value);  
    new IBinarySearchTree\<K, V\> Remove(K key);  
    IBinarySearchTree\<K, V\> Search(K key);  
}

But how on earth are we going to insert a node into a binary *search* tree? Since an immutable binary tree has got to be built "from the leaves", aren't we going to have to rebuild the entire tree every time we insert a node? (Without loss of generality, assume that the item is not in the tree already. We'll discuss what to do in that situation later.)

Let's think about it. If the tree is empty, then insertion is trivial -- you just return a new single-node tree.

If the tree is not empty, then either the new element fits in the right subtree or the left subtree. Suppose it's the right. Then you do not have to rebuild any of the left subtree, only the root node and the right subtree. That eliminates half of the possible work right there\! But that then is true of the right subtree as well -- either it is trivial, or half of its work will be eliminated immediately. We keep on eliminating half the work every time, and 1/2 + 1/4 + 1/8 + ... gets pretty darn close to "all the work".

Awesome. Or, perhaps not.

Remember, it is perfectly legal to have a binary search tree where **every left subtree is empty**. You just have a long, long train of trees down the right hand side. Essentially what you have is the sorted stack that we had before. Inserting a new item at the bottom would require building the whole tree again. **Eliminating the** **left hand side only eliminates half the work if half the nodes are on the left hand side\!**

But suppose we had a way to guarantee that these sorts of pathological situations never happened. What then would the cost of inserting an item into a binary tree be? Clearly the cost of inserting the node is the cost of generating O(h) new tree nodes, where h is the depth at which the new node is inserted. Therefore, all we need to do in order to keep insertion relatively cheap is to keep the tree height as small as possible everywhere. Such a tree is called a **balanced binary tree**, and there are lots of ways to balance a binary tree. You can balance a tree by guaranteeing that the left side has some fraction of the total nodes (a "weight balanced tree") or that the height of the left side is less than some multiplier of the height of the right side, or that the left and right heights differ by less than some constant (a height balanced tree).

I don't want this to turn into a tutorial on tree balancing; there are plenty of web sites out there that can discuss the intricacies of tree rotations for height balancing. So tell you what, anyone who is interested, read the [wikipedia page on AVL tree balancing](http://en.wikipedia.org/wiki/AVL_tree), and next time on FAIC I'll post an implementation of an immutable height-balanced AVL tree which implements the contract above.

(This is pretty straightforward stuff; the immutable queue was slightly mind-blowing to a number of readers, but immutable balanced binary trees are just straightforward busywork. After we get through that I'll show you guys how to make a combination-stack-and-queue that can be efficiently read from **both** ends, *and* is immutable. It uses the generic type system in an interesting way to make it all work efficiently.)

