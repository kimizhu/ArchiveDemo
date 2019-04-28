# Immutability in C\# Part Nine: Academic? Plus my AVL tree implementation

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/21/2008 7:13:00 PM

-----

Good Monday morning all. I have received a lot of good comments on this series so far that I thought I would speak to a bit.

##### Academic?

First and foremost, a number of people have asked questions which could be summed up as "isn't this just an academic exercise? I have a job to do here\!"

No, I do not believe that it is just an academic exercise, not at all. When I compare the practical, solving-real-problems code I write using immutable or ("mostly" immutable objects) to that which uses highly mutable objects, I find that the solutions which use immutable objects are easier to reason about. Knowing that an object isn't going to be "edited" by some other bit of code later on means that anything I deduce about it now is going to continue to be true forever.

I also find that programming in a more "functional" style using immutable objects somehow encourages me to write very small, clear methods that each do one thing very well. This is certainly possible when writing code with mutable objects, but I find that something about writing code for immutable objects encourages that style more. I haven't managed to quite figure out why that is yet. I like that style of programming a lot. Take a look at the short little methods below, for, say, tree rotation. Each of them is about four lines long and obviously correct if you read it carefully. That gives you confidence that those rotation helpers can then be composed to make a correct balancer, which gives you confidence that the tree is balanced.

Another reason why immutable objects tend to be easier to reason about is that this style encourages non-nullable programming. Take a look at the code from the previous entries in this series. The keyword "null" does not appear in any of them\! And why should it? Why *should* we represent an empty stack the same way we represent an empty queue, the same way we represent an empty tree? By ensuring that everything which points to something else is never null, all that boring, slow, hard to read, hard to reason about code which checks to see if things are null just goes away. You need never worry whether that reference is null again if you design your objects so that there's never a null reference in the first place.

And as we'll see in later episodes of FAIC, immutable objects also make it easier to implement performance improvements such as memoization of complex calculations. If an object never changes then you can always cache it for reuse later. An object which someone can edit is less useful that way. Immutable objects stored in a stack make undo-redo operations trivial. And so on.

Learning how to use this style in your own code is therefore potentially quite handy. I believe it is also the case that more and more objects produced by other people will be immutable, so knowing how to deal with them will be increasingly important. For example, the objects which represent expression trees in C\# 3.0 are all immutable. If you want to take an expression tree and transform it, you're simply not going to be able to transform it "in place". You're going to have to build a new expression tree out of the old one; knowing how to do so efficiently will help.

##### Thread safe?

The second question I've gotten a lot is "are immutable objects really more thread safe?" Well, it depends on what you mean by "thread safe". Immutable objects are not necessarily "more" thread safe, but they're a different kind of thread safe.

Let's explore this a bit more. What exactly do we mean by "thread safe"? We can talk about lock order and deadlocks and all of those aspects of thread safety, but let's put those aside for a moment and just consider basic race conditions. Suppose you have two threads each enqueuing jobs onto a queue object, and a third thread dequeuing jobs. Normally we think of the queue as being "thread safe" if no matter what the timing of those three threads happens to be, there is no way that a job is ever "lost". Two enqueues which happen "at the same time" result in a queue with those two things on it, and at some point, both will be dequeued.

If the queue is immutable then implementing this scenario just moves the problem around; now it becomes about serializing access to the mutable variable which is being shared by all three threads. Immutable objects make this kind of scenario harder to reason about and implement; **better to just write a threadsafe mutable queue in the first place if that's the problem you're trying to solve.**

But let's think about this a different way. Having a threadsafe mutable queue means that you never have any accurate information whatsoever about the state of the queue\! You ask the queue "are you empty?" and it says "no", and you dequeue it and hey, you get an "empty queue" exception. Why? Because some other thread dequeued it in the time between when you asked and when you issued your dequeue request. You end up living in a world where enqueuing a queue can possibly make it shorter and dequeuing can make it longer depending on what is going on with other threads. It becomes impossible to reason locally about operations on an object; your consciousness has to expand to encompass all possible operations on the object. It's this necessity for global understanding that makes multithreaded programming so error-prone and difficult.

Immutable queues, on the other hand, give you complete thread safety in this regard. You enqueue an element onto a particular immutable queue and the result you get back is always the same no matter what is happening to that queue on any other thread. You ask a queue if it is empty, if it says no, then you have a 100% iron-clad guarantee that you can safely dequeue it. You can share data around threads as much as you like without worrying that some other thread is going to make an edit which messes up your logic. You can take a chunk of data and run ten different analyzers over it on ten different threads, each one making "changes" to the data during its analysis, and never interfering with each other.

##### AVL Tree implementation

As promised last time, my supremely unexciting implementation of a self-balancing height-balanced immutable AVL tree in C\#. Note that I have made the choice to cache the height in every node rather than recalculating it. That trades memory usage for a bit of extra speed, which is probably worth it in this case. (Though of course to truly answer the question we'd want to set goals, try it both ways and measure the results.)

This is all pretty straightforward. Next time I want to start to wrap up this series by looking at a tricky data structure that solves the problem of building an immutable double-ended queue.

     public sealed class AVLTree\<K, V\> : IBinarySearchTree\<K, V\> where K : IComparable\<K\>  
    {  
        private sealed class EmptyAVLTree : IBinarySearchTree\<K, V\>  
        {  
            // IBinaryTree  
            public bool IsEmpty { get { return true; } }  
            public V Value { get { throw new Exception("empty tree"); } }  
            IBinaryTree\<V\> IBinaryTree\<V\>.Left { get { throw new Exception("empty tree"); } }  
            IBinaryTree\<V\> IBinaryTree\<V\>.Right { get { throw new Exception("empty tree"); } }  
            // IBinarySearchTree  
            public IBinarySearchTree\<K, V\> Left { get { throw new Exception("empty tree"); } }  
            public IBinarySearchTree\<K, V\> Right { get { throw new Exception("empty tree"); } }  
            public IBinarySearchTree\<K, V\> Search(K key) { return this; }  
            public K Key { get { throw new Exception("empty tree"); } }  
            public IBinarySearchTree\<K, V\> Add(K key, V value) { return new AVLTree\<K, V\>(key, value, this, this); }  
            public IBinarySearchTree\<K, V\> Remove(K key) { throw new Exception("Cannot remove item that is not in tree."); }  
            // IMap  
            public bool Contains(K key) { return false; }  
            public V Lookup(K key) { throw new Exception("not found"); }  
            IMap\<K, V\> IMap\<K, V\>.Add(K key, V value) { return this.Add(key, value); }  
            IMap\<K, V\> IMap\<K, V\>.Remove(K key) { return this.Remove(key); }  
            public IEnumerable\<K\> Keys { get { yield break; } }  
            public IEnumerable\<V\> Values { get { yield break; } }  
            public IEnumerable\<KeyValuePair\<K,V\>\> Pairs { get { yield break; } }  
        }  
        private static readonly EmptyAVLTree empty = new EmptyAVLTree();  
        public static IBinarySearchTree\<K, V\> Empty { get { return empty; } }  
        private readonly K key;  
        private readonly V value;  
        private readonly IBinarySearchTree\<K, V\> left;  
        private readonly IBinarySearchTree\<K, V\> right;  
        private readonly int height;  
        private AVLTree(K key, V value, IBinarySearchTree\<K, V\> left, IBinarySearchTree\<K, V\> right)  
        {  
            this.key = key;  
            this.value = value;  
            this.left = left;  
            this.right = right;  
            this.height = 1 + Math.Max(Height(left), Height(right));  
        }  
        // IBinaryTree  
        public bool IsEmpty { get { return false; } }  
        public V Value { get { return value; } }  
        IBinaryTree\<V\> IBinaryTree\<V\>.Left { get { return left; } }  
        IBinaryTree\<V\> IBinaryTree\<V\>.Right { get { return right; } }  
        // IBinarySearchTree  
        public IBinarySearchTree\<K, V\> Left { get { return left; } }  
        public IBinarySearchTree\<K, V\> Right { get { return right; } }  
        public IBinarySearchTree\<K, V\> Search(K key)  
        {  
            int compare = key.CompareTo(Key);  
            if (compare == 0)  
                return this;  
            else if (compare \> 0)  
                return Right.Search(key);  
            else  
                return Left.Search(key);  
        }  
        public K Key { get { return key; } }  
        public IBinarySearchTree\<K, V\> Add(K key, V value)  
        {  
            AVLTree\<K, V\> result;  
            if (key.CompareTo(Key) \> 0)  
                result = new AVLTree\<K, V\>(Key, Value, Left, Right.Add(key, value));  
            else  
                result = new AVLTree\<K, V\>(Key, Value, Left.Add(key, value), Right);  
            return MakeBalanced(result);  
        }  
        public IBinarySearchTree\<K, V\> Remove(K key)  
        {  
            IBinarySearchTree\<K, V\> result;  
            int compare = key.CompareTo(Key);  
            if (compare == 0)  
            {  
                // We have a match. If this is a leaf, just remove it  
                // by returning Empty.  If we have only one child,  
                // replace the node with the child.  
                if (Right.IsEmpty && Left.IsEmpty)  
                    result = Empty;  
                else if (Right.IsEmpty && \!Left.IsEmpty)  
                    result = Left;  
                else if (\!Right.IsEmpty && Left.IsEmpty)  
                    result = Right;  
                else  
                {  
                    // We have two children. Remove the next-highest node and replace  
                    // this node with it.  
                    IBinarySearchTree\<K, V\> successor = Right;  
                    while (\!successor.Left.IsEmpty)  
                        successor = successor.Left;  
                    result = new AVLTree\<K, V\>(successor.Key, successor.Value, Left, Right.Remove(successor.Key));  
                }  
            }  
            else if (compare \< 0)  
                result = new AVLTree\<K, V\>(Key, Value, Left.Remove(key), Right);  
            else  
                result = new AVLTree\<K, V\>(Key, Value, Left, Right.Remove(key));  
            return MakeBalanced(result);  
        }  
        // IMap  
        public bool Contains(K key) { return \!Search(key).IsEmpty; }  
        IMap\<K, V\> IMap\<K, V\>.Add(K key, V value) { return this.Add(key, value); }  
        IMap\<K, V\> IMap\<K, V\>.Remove(K key) { return this.Remove(key); }  
        public V Lookup(K key)  
        {  
            IBinarySearchTree\<K, V\> tree = Search(key);  
            if (tree.IsEmpty)  
                throw new Exception("not found");  
            return tree.Value;  
        }  
        public IEnumerable\<K\> Keys { get { return from t in Enumerate() select t.Key; } }  
        public IEnumerable\<V\> Values { get { return from t in Enumerate() select t.Value; }  
        }  
        public IEnumerable\<KeyValuePair\<K,V\>\> Pairs  
        {  
            get  
            {  
                return from t in Enumerate() select new KeyValuePair\<K, V\>(t.Key, t.Value);  
            }  
        }  
        private IEnumerable\<IBinarySearchTree\<K, V\>\> Enumerate()  
        {  
            var stack = Stack\<IBinarySearchTree\<K,V\>\>.Empty;  
            for (IBinarySearchTree\<K,V\> current = this; \!current.IsEmpty || \!stack.IsEmpty; current = current.Right)  
            {  
                while (\!current.IsEmpty)  
                {  
                    stack = stack.Push(current);  
                    current = current.Left;  
                }  
                current = stack.Peek();  
                stack = stack.Pop();  
                yield return current;  
            }  
        }  
        // Static helpers for tree balancing  
        private static int Height(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.IsEmpty)  
                return 0;  
            return ((AVLTree\<K,V\>)tree).height;  
        }  
        private static IBinarySearchTree\<K, V\> RotateLeft(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.Right.IsEmpty)  
                return tree;  
            return new AVLTree\<K, V\>( tree.Right.Key, tree.Right.Value,  
                new AVLTree\<K, V\>(tree.Key, tree.Value, tree.Left, tree.Right.Left),  
                tree.Right.Right);  
        }  
        private static IBinarySearchTree\<K, V\> RotateRight(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.Left.IsEmpty)  
                return tree;  
            return new AVLTree\<K, V\>( tree.Left.Key, tree.Left.Value, tree.Left.Left,  
                new AVLTree\<K, V\>(tree.Key, tree.Value, tree.Left.Right, tree.Right));  
        }  
        private static IBinarySearchTree\<K, V\> DoubleLeft(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.Right.IsEmpty)  
                return tree;  
            AVLTree\<K, V\> rotatedRightChild = new AVLTree\<K, V\>(tree.Key, tree.Value, tree.Left, RotateRight(tree.Right));  
            return RotateLeft(rotatedRightChild);  
        }  
        private static IBinarySearchTree\<K, V\> DoubleRight(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.Left.IsEmpty)  
                return tree;  
            AVLTree\<K, V\> rotatedLeftChild = new AVLTree\<K, V\>(tree.Key, tree.Value, RotateLeft(tree.Left), tree.Right);  
            return RotateRight(rotatedLeftChild);  
        }  
        private static int Balance(IBinarySearchTree\<K, V\> tree)  
        {  
            if (tree.IsEmpty)  
                return 0;  
            return Height(tree.Right) - Height(tree.Left);  
        }  
        private static bool IsRightHeavy(IBinarySearchTree\<K, V\> tree) { return Balance(tree) \>= 2; }  
        private static bool IsLeftHeavy(IBinarySearchTree\<K, V\> tree) { return Balance(tree) \<= -2; }  
        private static IBinarySearchTree\<K, V\> MakeBalanced(IBinarySearchTree\<K, V\> tree)  
        {  
            IBinarySearchTree\<K, V\> result;  
            if (IsRightHeavy(tree))  
            {  
                if (IsLeftHeavy(tree.Right))  
                    result = DoubleLeft(tree);  
                else  
                    result = RotateLeft(tree);  
            }  
            else if (IsLeftHeavy(tree))  
            {  
                if (IsRightHeavy(tree.Left))  
                    result = DoubleRight(tree);  
                else  
                    result = RotateRight(tree);  
            }  
            else  
                result = tree;  
            return result;  
        }  
    }  
}

