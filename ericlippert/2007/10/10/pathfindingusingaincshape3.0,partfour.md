# Path Finding Using A\* in C\# 3.0, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2007 7:17:00 AM

-----

Finally we are ready to translate our pseudocode into C\# 3.0. 

What do we need to make the algorithm run? We need a start node, a destination node, a function which tells us the exact distance between two neighbours, and a function which tells us the estimated distance between the last node on a proposed path and the destination node.

We also need the ability to take any node and get a list of its neighbours. We'll represent that by an interface:

 

interface IHasNeighbours\<N\>   
{  
    IEnumerable\<N\> Neighbours { get; }  
}

Recall that the pseudocode looked like this:

 

closed = {}  
q = emptyqueue;  
q.enqueue(0.0, makepath(start))  
while q is not empty  
    p = q.dequeueCheapest  
    if closed contains p.last then continue;  
    if p.last == destination then return p  
    closed.add(p.last)  
    foreach n in p.last.neighbours  
        newpath = p.continuepath(n)  
        q.enqueue(newpath.TotalCost + estimateCost(n, destination), newpath)  
return null

And now we're all set. The translation from pseudocode to real code is quite straightforward:

 

static public Path\<Node\> FindPath\<Node\>(  
    Node start,  
    Node destination,  
    Func\<Node, Node, double\> distance,  
    Func\<Node, double\> estimate)  
    where Node : IHasNeighbours\<Node\>  
{  
    var closed = new HashSet\<Node\>();  
    var queue = new PriorityQueue\<double, Path\<Node\>\>();  
    queue.Enqueue(0, new Path\<Node\>(start));  
    while (\!queue.IsEmpty)  
    {  
        var path = queue.Dequeue();  
        if (closed.Contains(path.LastStep))  
            continue;  
        if (path.LastStep.Equals(destination))  
            return path;  
        closed.Add(path.LastStep);  
        foreach(Node n in path.LastStep.Neighbours)  
        {  
            double d = distance(path.LastStep, n);  
            var newPath = path.AddStep(n, d);  
            queue.Enqueue(newPath.TotalCost + estimate(n), newPath);  
        }  
    }  
    return null;  
}

**UPDATE**: I’ve been asked to clarify why the estimation function takes only one parameter. You could certainly declare the parameter as “Func\<Node, Node, double\> estimate”, and then call estimate(n, destination) if you’d rather. I prefer to use partial application when possible. If I have an estimation function that takes two nodes, then in the caller I would simply pass n=\>estimate(n, destination) as the estimation function. Either way works; it’s a stylistic thing.

You might wonder why we check to see if the current-best-path ends in a "closed" node after the dequeue, rather than testing that before we do the enqueue. Think about it this way: suppose we take the best path off the queue, note that its end node is "closed" so that we don't come back to it, and put all of its extensions on the queue. There might be at present a number of other paths which end in that node on the queue. All of them must be, by definition, worse than the path we just took off the priority queue. We should therefore not even consider them as possbilities anymore. If we happen to put more paths with that property on the queue, that's unfortunate. But when we eventually get to them we'll discard them automatically so its not a big deal.

A nice property of the A\* algorithm is that it finds the optimal path in a reasonable amount of time provided that:

  - the estimating function never overestimates the distance to the goal. (Think about what happens if the estimating function sometimes overestimates the distance; if the optimal path is one of the ones overestimated then it will possibly not make it to the front of the priority queue before a nonoptimal solution is found.)

  - calling the estimating function does not take very long.

One way to ensure that the estimating function never overestimates the distance is to always estimate zero. If you do so then this becomes Dijkstra's algorithm. However, the better your estimating function can get without going over (this really should have been called the "The Price Is Right" algorithm...) the faster this will identify the truly optimal path.

Unfortunately, the space complexity of this algorithm can be really quite high on complicated graphs. There are more complex versions of this algorithm which can deal with the space complexity, but I'm not going to go there.

I have a nice example of using A\* to find the shortest path around a lake (but not the [longest shortest path](http://blogs.msdn.com/ericlippert/archive/2004/12/15/whidbey-island-and-bagel-mathematics.aspx)\!) but I'm not going to post it here until Orcas ships. I'll just put up the whole project file at that time and you can check it out all at once.

Next time: at long last, Eric talks about variance in C\#.

