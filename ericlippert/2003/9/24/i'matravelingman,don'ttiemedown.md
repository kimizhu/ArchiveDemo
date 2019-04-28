# I'm a traveling man, don't tie me down

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/24/2003 11:24:00 PM

-----

While I was waiting for a shuttle the other day (Microsoft has a fleet of shuttle busses that take people all over the campuses) I was thinking about optimization heuristics.

Often we want to find an algorithm that determines the best way to do something. For example, the traveling salesman problem: given a set of cities and the cost of driving between all of them, what is the cheapest route that visits every city at least once? Such problems can be extremely difficult to solve\! But often we miss out on the fact that we don't need to find the optimal solution, we need to find the "good enough" solution. Or we need to find a solution that has **bounded inoptimality**. Often finding the best solution is impractical, but it would be nice to know that you can find a solution that doesn't suck *too* bad.

For the traveling salesman problem, for instance, you can easily construct the **minimum spanning tree** of the graph. Then the route becomes trivial: pick any node as the root, do a depth-first traversal of the spanning tree and go back to the root. You'll visit every edge twice and end up where you started from. The total cost is **twice** the cost of traversing the spanning tree **once**, and obviously the cost of the optimal solution can't possibly be *better* than the cost of traversing the minimum spanning tree once.

This doesn't give you an optimal solution (unless your graph is already a tree, of course) but it does give you a solution that is guaranteed to be between 100% and 200% of the optimal solution, and that might be good enough.

This sort of 200% heuristic crops up all the time. Do you rent DVDs or buy them? Suppose a DVD costs $24 to buy and $6 to rent. Obviously the optimal solution is to buy it if you are going to watch it four or more times, rent otherwise -- but that requires perfect knowledge of the future.

Can we come up with an algorithm that minimizes the maximum suckiness, given no information about the future? Well, the worst possible situation is buying a DVD and watching it once -- you just overpaid by a factor of four. Actually, no, a worse situation still is renting the DVD so many times that you end up paying way more than the purchase price.

The solution? Rent the DVD the first four times, and then the fifth time, buy it. No matter how many times you watch it, the worst you can do is pay 200% of the optimal price and typically you'll do a lot better. (Particularly if you can bring to bear more information, such as being able to forecast the number of future viewings.)

This same heuristic applies to waiting for shuttles, which is why I was thinking about this in the first place. How long should you wait before you give up and walk? If the shuttle is going to be right there 30 seconds after you call, it's foolish to walk. But it is also foolish to wait ten times as long as it would have taken you to walk.

I waited as long as it would have taken me to walk, and then I walked. Fortunately it was a nice day yesterday.

