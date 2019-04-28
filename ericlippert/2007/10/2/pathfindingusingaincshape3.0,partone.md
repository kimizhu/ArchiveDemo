# Path Finding Using A\* in C\# 3.0, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/2/2007 7:31:00 PM

-----

As we get into the home stretch for the Orcas release I’ve been spending a little time lately just playing around implementing some “classic” algorithms and data structures in C\#. The one I implemented today is the famous A\* algorithm.

Surely you’ve played games like Age of Empires. You know the kind of games I mean. There are a bunch of little dudes walking around a map containing barriers (mountains, walls, etc) and somehow the game’s pseudo-intelligence finds the shortest possible path for the guys to walk from point A to point B. Most games do this with a surprisingly straightforward algorithm.

Suppose you’ve got a bunch of “nodes”. Say, intersections in a city. Each node has a bunch of “neighbours” – intersections that can be immediately reached by locomoting down a particular street involved in the intersection. Each “edge” (ie, street) between nodes has a “cost” – the amount of time it takes to travel from one node to the neighbour on that street. Perhaps the cost is based on its length, traffic density, etc.

A “path from A to B” consists of a list of nodes starting with A, ending with B, where every step along the path is from neighbour to neighbour. The total cost of a path is obviously the sum of all the costs of every transition from neighbour to neighbour on the path.

Let’s start by considering a really stupid, awful, buggy path finding algorithm, and then we’ll make it better.

 

q = emptyqueue  
q.enqueue(makepath(start))  
while q is not empty  
    p = q.dequeue  
    if p.last == destination then return p  
    foreach n in p.last.neighbours  
        q.enqueue(p.continuepath(n))  
// failed to find path.  
return null

What’s wrong with this picture?

Well, suppose we have four nodes,arranged like this: A \<--\> C \<--\> D \<--\> B. Does this algorithm find the obvious path from A to B?

Run the code in your head: it starts by putting the path “A” on the queue. Then it takes it off and puts “A-C” on the queue. Then it takes it off and puts “A-C-A” and “A-C-D” on the queue. Then it takes “A-C-A” off the queue and puts “A-C-A-C” on the queue... OK, eventually we get there, but we seem to be spending a lot of time going in circles.  And if D does not actually connect to B, if you can't get there from here, then we really *do* go into an infinite loop.

Clearly this is all screwed up.  We need to disallow paths which loop back on themselves, because those are clearly not going to be the best possible path. We're at best wasting time looking at them, and at worst going into an infinite loop that chews up all the memory in the system.

Let’s fix that up. We’ll create a “closed” set, ie, the set of “nodes we have already considered, so do not loop back to them.”

 

closed = {}  
q = emptyqueue;  
q.enqueue(makepath(start))  
while q is not empty  
    p = q.dequeue  
    if closed contains p.last then continue  
    if p.last == destination then return p  
    closed.add(p.last)  
    foreach n in p.last.neighbours   
        q.enqueue(p.continuepath(n))  
return null

This is *better*, in that it actually terminates in a finite amount of time. But my goodness, is this algorithm ever slow. Essentially the algorithm is “At every intersection, check to see if you're at your destination. If not, pick a direction – any direction\! - and walk as far as you can like that without revisiting your steps. If you ever run out of options, backtrack until you get to an intersection that goes somewhere you haven’t tried yet. Keep doing that until either you get to your destination or you have tried every possible intersection.”

I guarantee you that if you try that in a reasonably sized city, you will not get from point A to point B in any kind of reasonable time. And when you do succeed, the path which you come up with may well be absolutely crazy. Obviously this is not how people navigate between points in cities.

The problem is that we are picking which neighbour to try next essentially at random. We can do better than that. **We want to direct our search so that the path we spend most of our time concentrating on is the path which is most likely to lead us to the goal.**

Now, that sounds like a bit of a chicken-and-egg problem. Finding the best path is precisely what we are trying to do, so how can we determine which path is the one we want to be concentrating on?

We shall have to guess. We will rank all potential paths based on the sum of two factors:

  - First, what is the total cost of each potential path so far? This is *known*.
  - Second, what is the estimated cost of getting from the end of each potential path to the goal? This is a *guess*.

And now our algorithm becomes:

 

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

which is the famous A\* algorithm\! Surprisingly straightforward, eh?

Next time: actually implementing this thing in C\# 3.0.

