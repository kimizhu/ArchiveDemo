# Immutability in C\# Part Two: A Simple Immutable Stack

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/4/2007 10:26:00 AM

-----

Let’s start with something simple: an immutable stack.

Now, immediately I hear the objection: *a stack is by its very nature something that changes.* A stack is an abstract data type with the interface

 

void Push(T t);  
T Pop();  
bool IsEmpty { get; }

You push stuff onto it, you pop stuff off of it, it changes. How can it be immutable?

Every time you need to make a data structure immutable, you use basically the same trick: an operation which “changes” the data structure does so by constructing a new data structure. The original data structure stays the same.

How can that possibly be efficient? Surely we’ll be allocating memory all over the place\! Well, actually, in this case, no. An immutable stack is every bit as efficient as a mutable stack. Even better: in some cases, it can be considerably more efficient, as we'll see.

Let’s start by defining an interface for our immutable structure. While we’re at it, we’ll fix a problem with the stack ADT above, namely that you cannot interrogate the stack without changing it. And we’ll make the stack enumerable just for the heck of it:

 

    public interface IStack\<T\> : IEnumerable\<T\>  
    {  
        IStack\<T\> Push(T value);  
        IStack\<T\> Pop();  
        T Peek();  
        bool IsEmpty { get; }  
    }

Pushing and popping give you back an entirely new stack, and Peek lets you look at the top of the stack without popping it.

Now let’s think about constructing one of these things here. Clearly if we have an existing stack we can construct a new one by pushing or popping it. But we have to start somewhere. Since every empty stack is the same, it seems sensible to have a singleton empty stack.

 

    public sealed class Stack\<T\> : IStack\<T\>  
    {  
        private sealed class EmptyStack : IStack\<T\>  
        {  
            public bool IsEmpty { get { return true; } }  
            public T Peek() { throw new Exception("Empty stack"); }  
            public IStack\<T\> Push(T value) { return new Stack\<T\>(value, this); }  
            public IStack\<T\> Pop() { throw new Exception("Empty stack"); }  
            public IEnumerator\<T\> GetEnumerator() { yield break; }  
            IEnumerator IEnumerable.GetEnumerator() { return this.GetEnumerator(); }  
        }  
        private static readonly EmptyStack empty = new EmptyStack();  
        public static IStack\<T\> Empty { get { return empty; } }  
        private readonly T head;  
        private readonly IStack\<T\> tail;  
        private Stack(T head, IStack\<T\> tail)  
        {  
            this.head = head;  
            this.tail = tail;  
        }  
        public bool IsEmpty { get { return false; } }  
        public T Peek() { return head; }  
        public IStack\<T\> Pop() { return tail; }  
        public IStack\<T\> Push(T value) { return new Stack\<T\>(value, this); }  
        public IEnumerator\<T\> GetEnumerator()  
        {  
            for(IStack\<T\> stack = this; \!stack.IsEmpty ; stack = stack.Pop())  
                yield return stack.Peek();  
        }  
        IEnumerator IEnumerable.GetEnumerator() {return this.GetEnumerator();}  
    }

And now we can easily create stacks and push stuff onto them. Notice how the fact that we have immutability means that stacks with the same tail can share state, saving on memory:

 

IStack\<int\> s1 = Stack\<int\>.Empty;  
IStack\<int\> s2 = s1.Push(10);  
IStack\<int\> s3 = s2.Push(20);  
IStack\<int\> s4 = s2.Push(30); // shares its tail with s3.

Painless. Next time: a variant variation on this theme.

