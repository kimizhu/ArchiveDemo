# Path Finding Using A\* in C\# 3.0, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/4/2007 10:33:00 AM

-----

In order to [implement the A\* algorithm in C\# 3.0](http://blogs.msdn.com/ericlippert/archive/2007/10/02/path-finding-using-a-in-c-3-0.aspx) I am going to need to implement some custom data structures. Today we’ll consider how to implement the “path”.

You’ll notice in my description of the algorithm that the only operations we perform on paths are:

  - Make a new path consisting of a single node.
  - Make a new path by adding a new node onto the end of an existing path.
  - Fetch the last element on the path.
  - Get the total cost of a path.

This cries out to me “Eric\! A path is an immutable stack\!”

A mutable stack like System.Collections.Generic.Stack\<T\> is clearly not suitable. We want to be able to take an existing path and create new paths from it for all of the neighbours of its last element, but **pushing a new node onto the standard stack modifies the stack**. We’d have to make copies of the stack before pushing it, which is silly because then we’d be duplicating all of its contents unnecessarily.

Immutable stacks do not have this problem. Pushing onto an immutable stack merely creates a brand-new stack which links to the old one as its tail. Since the stack is immutable, there is no danger of some other code coming along and messing with the tail contents. You can keep on using the old stack to your heart’s content.

ASIDE: **Immutable data structures are the way of the future in C\#.** It is much easier to reason about a data structure if you know that it will never change. Since they cannot be modified, they are automatically threadsafe. Since they cannot be modified, you can maintain a stack of past “snapshots” of the structure, and suddenly undo-redo implementations become trivial. On the down side, they do tend to chew up memory, but hey, that’s what garbage collection was invented for, so don’t sweat it. I’ll be talking more about programming using immutable data structures in this space over the next few months.

Let’s make an immutable stack of nodes which tracks the total cost of the whole path. Later on, we’ll see that we need to enumerate the nodes of this thing, so we’ll do a trivial implementation of IEnumerable while we’re at it. And since we don’t know exactly what a node will look like, let’s make the thing generic:

 

class Path\<Node\> : IEnumerable\<Node\>  
{  
    public Node LastStep { get; private set; }  
    public Path\<Node\> PreviousSteps { get; private set; }  
    public double TotalCost { get; private set; }  
    private Path(Node lastStep, Path\<Node\> previousSteps, double totalCost)  
    {  
        LastStep = lastStep;  
        PreviousSteps = previousSteps;  
        TotalCost = totalCost;  
    }  
    public Path(Node start) : this(start, null, 0) {}  
    public Path\<Node\> AddStep(Node step, double stepCost)  
    {  
        return new Path\<Node\>(step, this, TotalCost + stepCost);  
    }  
    public IEnumerator\<Node\> GetEnumerator()  
    {  
        for (Path\<Node\> p = this; p \!= null; p = p.PreviousSteps)  
            yield return p.LastStep;  
    }  
    IEnumerator IEnumerable.GetEnumerator()  
    {  
        return this.GetEnumerator();  
    }  
}

Well, that was painless. Next time: implementing the priority queue.

