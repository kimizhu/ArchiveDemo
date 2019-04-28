# Immutability in C\# Part 10: A double-ended queue

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/22/2008 10:15:53 AM

-----

Based on the comments, the implementation of a single-ended queue as two stacks was somewhat mind-blowing for a number of readers. People, you ain't seen nothing yet.

Before we get into the actual bits and bytes of the solution, think for a bit about how you might implement an immutable queue which could act like both a stack or a queue at any time. You can think of a stack as "it goes on the left end, it comes off the left end", and a queue as "it goes on the left end, it comes off the right end". Now we want "it goes on and comes off either end". For short, we'll call a double ended queue a "deque" (pronounced "deck"), and give our immutable deque this interface:

 

    public interface IDeque\<T\>  
    {  
        T PeekLeft();  
        T PeekRight();  
        IDeque\<T\> EnqueueLeft(T value);  
        IDeque\<T\> EnqueueRight(T value);  
        IDeque\<T\> DequeueLeft();  
        IDeque\<T\> DequeueRight();  
        bool IsEmpty { get; }  
    }

##### Attempt \#1

We built a single-ended queue out of two stacks. Can we pull a similar trick here?  How about we have the "left stack" and the "right stack". Enqueuing on the left pushes on the left stack, enqueuing on the right pushes on the right stack, and so on.

Unfortunately, this has some problems. What if you are dequeuing on the right and you run out of items on the right-hand stack?  Well, no problem, we'll pull the same trick as before -- reverse the left stack and swap it with the right stack.

The trouble with that is, suppose the left stack is { 1000, 999, ..., 3, 2, 1 } and the right stack is empty. Someone dequeues the deque on the right. We reverse the stack, swap them and pop the new right stack. Now we have an empty left-hand stack and { 2, 3, 4, .... 1000 } on the right hand stack. It took 1000 steps to do this. Now someone tries to dequeue on the left. We reverse the right queue, swap, and pop, and now we have { 999, 998, ... 3, 2 }.  That took 999 steps. If we keep on dequeuing alternating on the right and left we end up doing on average **five hundred pushes per step**. That's terrible performance. Clearly this is an O(n<sup>2</sup>) algorithm.

##### Attempt \#2

Our attempt to model this as a pair of stacks seems to be failing. Let's take a step back and see if we can come up with a recursively defined data structure which makes it more apparent that there is **cheap access to each end.**

The standard recursive definition of a stack is "a stack is either empty, or an item (the head) followed by a stack (the tail)". It seems like we ought to be able to say "a deque is either empty, or an item (the left) followed by a deque (the middle) followed by an item (the right)".

Perhaps you have already seen the problem with this definition; a deque by this definition always has an even number of elements\! But we can fix that easily enough.  A deque is:

1\) empty, or  
2\) a single item, or  
3\) a left item followed by a middle deque followed by a right item.

Awesome. Let's implement it.

 

    // WARNING: THIS IMPLEMENTATION IS AWFUL. DO NOT USE THIS CODE.  
    public sealed class Deque\<T\> : IDeque\<T\>  
    {  
        private sealed class EmptyDeque : IDeque\<T\>  
        {  
            public bool IsEmpty { get { return true; } }  
            public IDeque\<T\> EnqueueLeft(T value) { return new SingleDeque(value); }  
            public IDeque\<T\> EnqueueRight(T value) { return new SingleDeque(value); }  
            public IDeque\<T\> DequeueLeft() { throw new Exception("empty deque"); }  
            public IDeque\<T\> DequeueRight() { throw new Exception("empty deque"); }  
            public T PeekLeft () { throw new Exception("empty deque"); }  
            public T PeekRight () { throw new Exception("empty deque"); }  
        }  
        private sealed class SingleDeque : IDeque\<T\>  
        {  
            public SingleDeque(T t) { item = t; }  
            private readonly T item;  
            public bool IsEmpty { get { return false; } }  
            public IDeque\<T\> EnqueueLeft(T value) { return new Deque\<T\>(value, Empty, item); }  
            public IDeque\<T\> EnqueueRight(T value) { return new Deque\<T\>(item, Empty, value); }  
            public IDeque\<T\> DequeueLeft() { return Empty; }  
            public IDeque\<T\> DequeueRight() { return Empty; }  
            public T PeekLeft () { return item; }  
            public T PeekRight () { return item; }  
        }  
        private static readonly IDeque\<T\> empty = new EmptyDeque();  
        public static IDeque\<T\> Empty { get { return empty; } }  
        public bool IsEmpty { get { return false; } }  
        private Deque(T left, IDeque\<T\> middle, T right)  
        {  
            this.left = left;  
            this.middle = middle;  
            this.right = right;  
        }  
        private readonly T left;  
        private readonly IDeque\<T\> middle;  
        private readonly T right;  
        public IDeque\<T\> EnqueueLeft(T value)  
        {  
            return new Deque\<T\>(value, middle.EnqueueLeft(left), right);  
        }  
        public IDeque\<T\> EnqueueRight(T value)  
        {  
            return new Deque\<T\>(left, middle.EnqueueRight(right), value);  
        }  
        public IDeque\<T\> DequeueLeft()  
        {  
            if (middle.IsEmpty) return new SingleDeque(right);  
            return new Deque\<T\>(middle.PeekLeft(), middle.DequeueLeft(), right);  
        }  
        public IDeque\<T\> DequeueRight()  
        {  
            if (middle.IsEmpty) return new SingleDeque(left);  
            return new Deque\<T\>(left, middle.DequeueRight(), middle.PeekRight());  
        }  
        public T PeekLeft () { return left; }  
        public T PeekRight () { return right; }  
    }

I seem to have somewhat anticipated my denouement, but this is coding, not mystery novel writing. What is so awful about this implementation? It seems like a perfectly straightforward implementation of the abstract data type. But it turns out to be actually *worse* than the two-stack implementation we first considered. What are your thoughts on the matter?

Next time, what's wrong with this code and some groundwork for fixing it.

