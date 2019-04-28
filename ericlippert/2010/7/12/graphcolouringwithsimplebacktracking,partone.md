# Graph Colouring With Simple Backtracking, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/12/2010 8:15:00 AM

-----

As regular readers know, I'm interested in learning how to change my C\# programming style to emphasize more concepts from functional programming, like use of **immutable** rather than **mutable** data structures and use of **declarative** control flow like LINQ queries instead of **imperative** control flow in the form of loops. I thought I'd solve a fairly straightforward problem using a mix of immutable and mutable, declarative and imperative styles to indicate when each is useful and appropriate.

I picked for my example the problem of colouring a map so that no two regions that share a border have the same colour. (Note that we count two regions as sharing a border if they meet along a border *line*; regions that meet only at a *corner* are allowed to be the same colour.) This is a famous problem. It is clear that on a plane or sphere, you can find a map that *requires* at least four colours. Take a look at [a map of South America](http://www.mygeo.info/karten/802788.jpg), for example; clearly Brazil, Bolivia, Paraguay and Argentina all share a border with each other and therefore require at least four colours. What is not at all clear is the fact that given any map, you can always find a colouring that requires no more than four colours; [four colours is always enough on a plane or a sphere](http://en.wikipedia.org/wiki/Four_color_theorem).

Of course, this is of no interest whatsoever to real mapmakers, who have no shortage of colours to choose from, and who often have additional constraints. For example, if you coloured a political map of the world with only four colours then either Bolivia or Paraguay would have to be the same colour as the ocean. There are also situations in real maps where you want two things to be of the same colour, like Alaska and the lower 48 states of the United States, which could introduce additional constraints.

Even if the theoretical problem of map colouring is not germane to the practical problem of making a map of a real thing, the problem is still of both theoretical and practical interest. To solve it, we can generalize the problem from a problem about touching regions on a plane map to nodes in a graph. Make one node for each region. Any two regions which must be coloured differently from each other share an edge. If we can find a colouring that assigns colours to each node in the graph, then we have a colouring for the corresponding map. If we were trying to colour South America, for instance, we simply need to colour this graph:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/7711.SouthAmerica.png)

I've shown the graph with a proposed colouring that works; there are several other possibilities.

Let's start by designing a simple data structure to represent an arbitrary graph. Every design process is the result of making compromises against competing design goals, so I think it is helpful to list some of those competing goals and what tradeoffs we're considering.

There are two standard ways to represent an arbitrary graph with n nodes and e edges; either you make an n by n array of bools that says whether there is an edge between two nodes, or you make a list of size e that gives all the edges in no particular order.

The benefit of the former approach is that it is extremely easy to get a list of all the edges coming out of a given node; the down side is that it uses up a lot of space if n is large and e is small. The benefit of the latter is that it is space-efficient if the graph is sparse, but it is inefficient to answer the question "what are all the neighbours of this node?" **(Tradeoff: space vs time.)** In both cases, the naive approach wastes about 50% of the space if the graph is actually undirected, because an edge from A to B is also an edge from B to A.

However, for our purposes the graphs are going to be relatively small anyway, so I don't particularly care which approach we take or that we're wasting a few bytes here and there. Let's go with the former approach and say that a graph is represented internally as an n by n adjacency matrix.

Should the graph be a directed or undirected graph? For some relationships a directed graph is clearly the way to go; a family tree, for example, is clearly a *directed* graph. If Alice is the mother of Bob then Bob is not the mother of Alice\! It seems like the concept we are trying to represent, "neighbouring regions" is inherently *undirected*. It's not like Brazil is a neighbour of Peru but Peru is not a neighbour of Brazil.

We could make a directed graph and build an undirected graph out of it by always adding edges in pairs, or we could build an undirected graph from the get-go, or we could use a directed graph as the base class of an undirected graph, or we could make an undirected graph and make a directed graph out of it by decorating edges with direction information if we needed to. There's all kinds of things we could do.

Suppose we do the first one: simply build a directed graph and require the user to add edges in pairs if they want an undirected graph. This makes the data structure more flexible (by being able to represent both directed and undirected graphs) and more bug-prone (because a user that wants to represent an undirected graph is responsible for maintaining the invariant themselves; **any time you add redundancy to a data structure you add the potential for bugs**.)

Is it actually more bug prone for our purposes? Even though the adjacency graph is logically *undirected*, for our purposes it is perfectly valid for the graph to *actually* be directed. It doesn't matter whether those edges have arrows on them or not\! If somehow we represent that Brazil is a neighbour of Peru but accidentally say that Peru is not a neighbour of Brazil then *we still will discover that they cannot both be green*. Just knowing one direction is sufficient.

In short, we *could* create an undirected graph class that managed ensuring that edges always came in pairs, but for now at least we won't. **(Tradeoffs: flexibility vs simplicity, complexity (of having multiple classes to represent graphs) vs imposition of responsibility upon users.) ** 

// An immutable directed graph  
internal sealed class Graph  
{  
  // If there is an edge from A to B then neighbours\[A,B\] is true.  
  readonly private bool\[,\] neighbours;  
  readonly private int nodes;  
  public Graph(int nodes, IEnumerable\<Tuple\<int, int\>\> edges)  
  {  
    this.nodes = nodes;  
    this.neighbours = new bool\[nodes, nodes\];  
    foreach (var edge in edges)  
      this.neighbours\[edge.Item1, edge.Item2\] = true;  
  }  
  public IEnumerable\<int\> Neighbours(int node)  
  {  
    return  
      from i in Enumerable.Range(0, Size)  
      where this.neighbours\[node, i\]  
      select i;  
  }  
  public int Size { get { return this.nodes; } }  
}

Let's go through this from top to bottom.

I've made the class sealed. This means that I do not have to consider what implementation details must be exposed so that an arbitrary class can extend this code effectively and efficiently. I do not have any scenarios in which this code needs to be extended. By the You Ain't Gonna Need It (YAGNI) principle I'm going to design the code so that it cannot be extended. [Designing for safe and effective reuse via inheritance is difficult and I don't want to take on that expense if I don't have to](http://blogs.msdn.com/b/ericlippert/archive/2004/01/22/61803.aspx). Nor do I want to make an implicit promise that I cannot deliver on; an unsealed class basically says to the user "go ahead and extend me, I promise I'll never break you". Implicitly taking on a commitment to preventing the [brittle base class failure](http://blogs.msdn.com/b/ericlippert/archive/tags/brittle+base+classes/) is not something I want to do. **(Tradeoff: potential code reuse vs increased design and maintenance costs).** Also, sealing is a breaking change but unsealing is not. If I seal now, I can always change my mind later. If I leave it unsealed then I cannot choose to seal it later without breaking any derived classes.

The class is internal, not public. I want this to be an implementation detail of my application, not a part of a general-purpose library. As we'll see, this decision has an effect on other implementation choices. **(Tradeoff: again, potential code reuse by others vs increased design and maintenance costs)**

The class name is Graph, not, say, DirectedGraph. If I ever wanted to make an UndirectedGraph class then I'd want to rename this one. But since in my application this will be the only graph class, I'm willing to have the simpler name. Note that I absolutely do not want to have the name of the class reflect its implementation details; this is not an AdjacencyMatrixGraph. I want the *name* to represent what it is logically, not what its implementation details are.

The class is not generic. Each node is an integer from 0 to size-1. We could have made this more complicated by allowing arbitrary data in a node; essentially making a generic "graph of T". But we can always maintain an external mapping from the integer of the node number to some other data if we like. **(Tradeoff: again, flexibility vs simplicity. Also, Single Responsibility Principle: this class represents graphs; let it do one thing well rather than making it into a graph and a map from nodes to data.)**

The graph internally uses an array for the adjacency matrix. We want data structures to be simple and efficient. Making a data structure that can be changed, either via mutation or by creating a new structure that incrementally re-uses parts of an old structure, adds flexibility and power but works against simplicity and efficiency. **(Tradeoff: flexibility vs simplicity and efficiency) ** For my purposes we're going to have all the information we need to make a graph all at once, create it, and be done with it. We don't need any way to add or remove nodes from a graph and make a new graph from it. **(YAGNI)** I therefore feel justified in using a mutable data structure, an array, as an implementation detail here, to make what is an entirely immutable structure from the point of view of the user. If we had scenarios where graphs were being edited then I'd probably choose a different approach to make an immutable graph.

The array is inefficient; we are wasting a lot of bits here. We could use an array of BitArrays. However, that adds complexity to the implementation and greatly slows down access to the bits; it is much faster to simply dereference a bool out of a two-dimensional array than it is to do all the bit-shifting logic to get a bit out of a BitArray. Without empirical data from a profiler that showed that the memory cost in reasonable scenarios was too high, I would stick with the simple bool array rather than the more complex bit vector solution.  **(Tradeoffs: code simplicity vs time efficiency vs space efficiency)**

We're also allocating a big two-dimensional array, which must be allocated in one monolithic block. If the number of nodes gets large, it might be difficult for the memory manager to find that much memory in a block. We could take on the additional memory expense of a BitArray solution or a ragged array solution, which would allow us to allocate n blocks of size n, rather than one block of size n by n. The former makes it much more likely that the memory manager will be able to fulfill our request, at the cost of the memory for logically "close" data being possibly spread out in memory, decreasing cache locality. **(Tradeoffs: again code simplicity vs time efficiency vs space efficiency)**

I redundantly store the size; we could compute the size by asking the array for its dimensions. If we were creating millions of graphs and were space-constrained then it might be reasonable to save those four bytes. But we're not. **(Tradeoffs: having to maintain redundancy vs code simplicity vs space efficiency)**

The fields are readonly, emphasizing that this type is immutable.

There is some redundancy in the information provided to the constructor. We could have deduced the number of nodes by taking the maximum integer found in the list of edges, instead of requiring that it be passed. Some of our options are:

(1) Iterate the sequence of edges twice, once to find the highest item, and then again to build the query. This is potentially expensive in time; we prefer to when possible build systems that only iterate a given sequence once because it could be representing a high-overhead database query. Even if the query overhead is cheap, it spends time iterating the sequence twice.  
(2) copy the sequence into an in-memory list. Search the copy for the highest item, and then build the graph out of it. This way potentially doubles both memory usage and time.  
(3) require that the user pass something other than an arbitrary sequence. This pushes the problem off to the user, and means that the entire sequence must be realized "eagerly" in memory, even if it could be calculated lazily.

Now, of course we know that we are going to be using n-squared memory and linear time anyway, so perhaps the memory and time concerns are not hugely relevant. But it seems reasonable that the user knows how many nodes there are in the graph ahead of time, since they're providing a list of edges, and therefore seems reasonable to require them to pass in this redundant information. **(Tradeoff: efficiency vs redundancy)**

We use a tuple type instead of a custom type to represent an edge. It seems reasonable to represent an edge as a pair of integers; there's no particular domain-specific semantics to an edge other than just a pair of nodes. So we'll just re-use a general-purpose data structure rather than needlessly inventing a new one that adds nothing. **(Tradeoff: cheap re-use of existing code vs having semantics clearly expressed in the type system)**

Note that we do not validate the data in the constructor. If the user says that the size is negative, or is enormous, or gives an edge that is out of bounds, then bad things happen like array-access-out-of-bounds exceptions. We could harden this system against ill use by reporting more specific exceptions. Or, we could assume that the user knows what they're doing and can diagnose the problem when they get an unexpected boneheaded exception. Were I designing this code for inclusing in the base class library, I absolutely positively would have all kinds of custom error detection and reporting in there; that's part of the "fit and finish" of a professional-grade library. If, on the other hand, this were an implementation detail of some analysis tool, a detail that was never publically exposed, then I'd just leave it like this and know that the maintainer of that code was responsible for using it correctly. **(Tradeoffs: code complexity vs "niceness" of reporting specific out-of-bounds errors).**

I've spelled "Neighbours" the British/Canadian way.  **(Tradeoff: correctness vs familiarity to Americans)**

Notice how the constructor uses a foreach loop whereas the Neighbours code returns a query object. That's deliberate; I want to use the loop to emphasize that the loop code is a *mechanism* that has a *side effect*. A looping statement calls that out very clearly. Fetching the neighbours of a particular node, on the other hand, is more like asking a question; what's important here is the *semantics*, and that the code is *free of side effects*. The query syntax emphasizes that.

What's interesting here is that this code is utterly trivial; it does almost nothing, and yet already we've made about a dozen design decisions involving subtle tradeoffs that will impact whether a given scenario can realistically be handled by this class.

Next time: representing a set of possible colours.

