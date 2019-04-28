# Immutability in C\# Part Four: An Immutable Queue

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/10/2007 10:04:00 AM

-----

An immutable queue is a bit trickier than an immutable stack, but we’re tough, we can handle it.

First off, the interface is straightforward; enqueuing or dequeuing results in a new queue:

 

    public interface IQueue\<T\> : IEnumerable\<T\>  
    {  
        bool IsEmpty { get; }  
        T Peek();  
        IQueue\<T\> Enqueue(T value);  
        IQueue\<T\> Dequeue();  
    }

But how ever are we to implement this? It’s not immediately obvious how to make an immutable first-in-first-out list.

The first insight we need is that a stack is essentially a “backwards” queue. Because I think it will come in handy later, I’m going to declare an extension method right now which takes a stack and reverses it:

 

        static public IStack\<T\> Reverse\<T\>(this IStack\<T\> stack )  
        {  
            IStack\<T\> r = Stack\<T\>.Empty;  
            for (IStack\<T\> f = stack; \!f.IsEmpty; f = f.Pop())  
              r = r.Push(f.Peek());         
            return r;  
        }

This is an O(n) operation for a stack of size n.

Now that we’ve got that we can make a first cut at our queue implementation. Let’s say that a queue is backed by a stack such that every time we need to get at the “dequeue end”, we’ll just reverse the stack:

 

sealed class Queue\<T\> : IQueue\<T\> {   
  private static readonly IQueue\<T\> empty = new Queue\<T\>(Stack\<T\>.Empty);  
  public static IQueue\<T\> Empty { return empty; }  
  private readonly IStack\<T\> backwards;  
  private Queue(IStack\<T\> backwards) { this.backwards = backwards; }  
  public T Peek() { return backwards.Reverse().Peek(); }  
  public IQueue\<T\> Enqueue(T t) { return new Queue\<T\>(backwards.Push(t)); }  
  public IQueue\<T\> Dequeue(T t) { return new Queue\<T\>(backwards.Reverse().Pop().Reverse()); }  
  public bool IsEmpty { get { return backwards.IsEmpty; } }  
  public IEnumerator\<T\> GetEnumerator() { return backwards.Reverse().GetEnumerator(); }  
  IEnumerator IEnumerable.GetEnumerator() {return this.GetEnumerator();}  
}

“Surely this is incredibly inefficient\!” I hear you say. Yes, most certainly it is\! Imagine enqueuing a thousand elements and then dequeuing them all. The thousand enqueues are each cheap, but the first dequeue requires us to entirely reverse a stack of 1000 items and then reverse another stack of 999 items.  The next dequeue requires reversing stacks of 999 and 998 items. Dequeing a single item costs O(n) in the size of the queue\! The total cost for all the dequeues is the reversal of 2000 stacks with an average size of about 500, so that’s about a million operations to dequeue the whole thing.

*When we reversed the stack to dequeue it the first time, we had a stack that was in the right order for the rest of the dequeues.* Why didn’t we keep that thing around? Only because the “forwards” stack that resulted is not in the right order for *enqueuing* new items. So let’s keep *two* stacks around, one in “forwards” order, ready to be dequeued, and one in “backwards” order, ready to be enqueued. If we go to dequeue and find that the forward stack is empty, only then will we reverse the backwards stack. (And let's make the empty queue a singleton, like we did with the empty stack.)

 

    public sealed class Queue\<T\> : IQueue\<T\>  
    {  
        private sealed class EmptyQueue : IQueue\<T\>  
        {  
            public bool IsEmpty { get { return true; } }  
            public T Peek () { throw new Exception("empty queue"); }  
            public IQueue\<T\> Enqueue(T value)  
            {  
                return new Queue\<T\>(Stack\<T\>.Empty.Push(value), Stack\<T\>.Empty);  
            }  
            public IQueue\<T\> Dequeue() { throw new Exception("empty queue"); }  
            public IEnumerator\<T\> GetEnumerator() { yield break; }  
            IEnumerator IEnumerable.GetEnumerator() { return this.GetEnumerator(); }  
        }  
        private static readonly IQueue\<T\> empty = new EmptyQueue();  
        public static IQueue\<T\> Empty { get { return empty; } }  
        public bool IsEmpty { get { return false; } }  
        private readonly IStack\<T\> backwards;  
        private readonly IStack\<T\> forwards;  
        private Queue(IStack\<T\> f, IStack\<T\> b)  
        {  
            forwards = f;  
            backwards = b;  
        }  
        public T Peek() { return forwards.Peek(); }  
        public IQueue\<T\> Enqueue(T value)  
        {  
            return new Queue\<T\>(forwards, backwards.Push(value));  
        }  
        public IQueue\<T\> Dequeue()  
        {  
            IStack\<T\> f = forwards.Pop();  
            if (\!f.IsEmpty)  
                return new Queue\<T\>(f, backwards);  
            else if (backwards.IsEmpty)  
                return Queue\<T\>.Empty;  
            else  
                return new Queue\<T\>(backwards.Reverse(), Stack\<T\>.Empty);  
        }  
        public IEnumerator\<T\> GetEnumerator()  
        {  
            foreach (T t in forwards) yield return t;  
            foreach (T t in backwards.Reverse()) yield return t;  
        }  
        IEnumerator IEnumerable.GetEnumerator() {return this.GetEnumerator(); }  
    }  
}

Isn’t this just as inefficient? No. Reason it through: every element (except the first one) is pushed on the backwards stack once, popped off the backwards stack once, pushed on the forwards stack once and popped off the forwards stack once. If you enqueue a thousand items and then dequeue two, yes, the second dequeue will be expensive as the backwards list is reversed, but the 998 dequeues following it will be cheap. That’s unlike our earlier implementation, where every dequeue was potentially expensive. Dequeuing is *on average* O(1), though it is O(n) for a single case.

Next time: OK, so we can do stacks and queues. Can we make an *immutable binary search tree with guaranteed good search performance*? But of course we can.

