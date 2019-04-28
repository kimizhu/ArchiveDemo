# Path Finding Using A\* in C\# 3.0, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2007 10:03:00 AM

-----

In order to make the A\* algorithm work we need to get the lowest-estimated-cost-path-discovered-so-far out of the list of paths under consideration. The standard data structure for doing so is called a “priority queue”. Priority queues are so-called because they are typically used to store a list of jobs where each job has an associated priority.

Now, there’s a bit of a semantic problem with priority queues in that we typically think of “priority 1” as being *higher* priority than “priority 2”, even though 1 is a *smaller* number than 2. Fortunately for us, that is *exactly* what we want in this case; the “highest priority” path is the one with the *least* estimated cost.

There are lots of ways to implement priority queues – I have a nice example of an immutable priority queue that I’ll probably blog about at some point, but for now, let’s go with an old-fashioned mutable priority queue.

A priority queue can be implemented as list of sub-queues with the list sorted in priority order. The implementation is pretty straightforward:

 

class PriorityQueue\<P, V\>  
{  
    private SortedDictionary\<P, Queue\<V\>\> list = new SortedDictionary\<P, Queue\<V\>\>();  
    public void Enqueue(P priority, V value)  
    {  
        Queue\<V\> q;  
        if (\!list.TryGetValue(priority, out q))  
        {  
            q = new Queue\<V\>();  
            list.Add(priority, q);  
        }  
        q.Enqueue(value);  
    }  
    public V Dequeue()  
    {  
        // will throw if there isn’t any first element\!  
        var pair = list.First();  
        var v = pair.Value.Dequeue();  
        if (pair.Value.Count == 0) // nothing left of the top priority.  
            list.Remove(pair.Key);  
        return v;  
    }  
    public bool IsEmpty  
    {  
        get { return \!list.Any(); }  
    }  
}

We could implement IEnumerable\<V\>, a Peek operation, etc, but this is all we need for A\*, so let’s not go crazy here.

Next time: now we have all the parts we need to implement A\*, so let's do it already.

