# Graph Colouring, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/26/2010 8:20:00 AM

-----

Let's give it a try. Can we colour South America with only four colours? Let's start by stating what all the edges are in the graph of South America:

const int Brazil = 0;  
const int FrenchGuiana = 1;  
const int Suriname = 2;  
const int Guyana = 3;  
const int Venezuala = 4;  
const int Colombia = 5;  
const int Ecuador = 6;  
const int Peru = 7;  
const int Chile = 8;  
const int Bolivia = 9;  
const int Paraguay = 10;  
const int Uruguay = 11;  
const int Argentina = 12;  
var SA = new Dictionary\<int, int\[\]\>()  
{  
    {Brazil, new\[\] { FrenchGuiana, Suriname, Guyana, Venezuala, Colombia, Peru, Bolivia, Paraguay, Uruguay, Argentina}},  
    {FrenchGuiana, new\[\] { Brazil, Suriname }},  
    {Suriname, new\[\] {Brazil, FrenchGuiana, Guyana}},  
    {Guyana, new\[\] {Brazil, Suriname, Venezuala}},  
    {Venezuala, new\[\] {Brazil, Guyana, Colombia}},  
    {Colombia, new \[\] {Brazil, Venezuala, Peru, Ecuador}},  
    {Ecuador, new\[\] {Colombia, Peru}},  
    {Peru, new\[\] {Brazil, Colombia, Ecuador, Bolivia, Chile}},  
    {Chile, new\[\] {Peru, Bolivia, Argentina}},  
    {Bolivia, new\[\] {Chile, Peru, Brazil, Paraguay, Argentina}},  
    {Paraguay, new\[\] {Bolivia, Brazil, Argentina}},  
    {Argentina, new\[\] {Chile, Bolivia, Paraguay, Brazil, Uruguay}},  
    {Uruguay, new\[\] {Brazil, Argentina}}  
};

We can transform this dictionary mapping nodes to neighbours into a list of edge tuples and build the graph:

var sagraph = new Graph(13, from x in SA.Keys  
                            from y in SA\[x\]  
                            select Tuple.Create(x, y));

Notice that this illustrates both a strength and a weakness of our earlier design decision to make the graph take a list of edges as its input. The graph could just as easily have taken a dictionary mapping nodes to neighbours, and then we wouldn't have to do this transformation in the first place\! It might have been a bad design decision. Such decisions always have to be made in the context of understanding how the user is actually going to use the type.

Anyway, now finding a four-colour solution is easy with our solver:

var sasolver = new Solver(sagraph, 4);  
var solution = sasolver.Solve();  
foreach (var colour in solution )  
    Console.WriteLine(colour);

 

We run this and get 0, 1, 2, 1, 2, 1, 0, 2, 0, 1, 2, 1, 3 for the colours. On the graph, that looks like this:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/4263.SouthAmerica.png)

 

Which is clearly a legal colouring, and manages to *almost* do it in three colours; only Argentina has to be the fourth. This of course is not a desirable property of real political maps; it would make Argentina stand out unnecessarily. But the way the algorithm priortizes hypothesizing lower colours over higher colours means that it is likely that in many real-world graphs, the higher colours will be rare with our solver.

Notice that the graph of South America is planar; it can be drawn without any of the edges crossing. In fact, all graphs that correspond to neighbouring regions on a plane or a sphere are planar graphs. (Do you intuitively see why?) However there is nothing stopping our solver from looking at more exotic graphs. Consider a torus, a doughnut shape. We can represent a torus on this flat screen by imagining that we "roll up" the square below so that the left edge meets the right edge, making a cylinder. And then bend the cylinder into a circle so that the the top edge meets the bottom edge:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/2318.Torus.png)

 Just to be clear: there are only seven regions on this rather bizarre map. All the regions with the same number that appear to be disjoint are connected because the torus "wraps around" to the opposite side, if you see what I mean.

Notice how on this map *every region touches every other region*, and therefore cannot be coloured with fewer than seven colours. The four-colour theorem does not hold on a torus\! (The seven-colour theorem does; you can colour any map that "wraps around on both sides" with seven or fewer colours.)

You can't graph this as a planar graph, so I'm not even going to try. The graph of this thing looks like this mess:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/0601.TorusGraph.png)

And indeed, if we fed this graph into our graph colourizer and asked for a four-colouring, it would not find one.

Let's call a subset of a graph where every node in the subset is connected to every other node in the subset a "fully connected subset". (In graph theory jargon that's called a [clique](http://en.wikipedia.org/wiki/Clique_\(graph_theory\)).) Clearly if any graph has a fully connected subset of size n then the graph requires at least n colours (and perhaps more). Graphs that have a lot of fully connected subsets are interesting to a lot of people. Next time we'll look at some particularly interesting graphs that have lots of fully connected subsets, throw our graph colourizer at them, and see what happens.

