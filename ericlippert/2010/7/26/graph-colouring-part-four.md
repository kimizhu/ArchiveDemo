<div id="page">

# Graph Colouring, Part Four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/26/2010 8:20:00 AM

-----

<div id="content">

<div class="mine">

Let's give it a try. Can we colour South America with only four colours? Let's start by stating what all the edges are in the graph of South America:

</div>

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Brazil = 0;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> FrenchGuiana = 1;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Suriname = 2;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Guyana = 3;</span>  
<span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"></span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Venezuala = 4;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Colombia = 5;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Ecuador = 6;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Peru = 7;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Chile = 8;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Bolivia = 9;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Paraguay = 10;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Uruguay = 11;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">const</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> <span style="COLOR: blue">int</span> Argentina = 12;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> SA = <span style="COLOR: blue">new</span> <span style="COLOR: #2b91af">Dictionary</span>\<<span style="COLOR: blue">int</span>, <span style="COLOR: blue">int</span>\[\]\>()  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">{  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Brazil, <span style="COLOR: blue">new</span>\[\] { FrenchGuiana, Suriname, Guyana, Venezuala, Colombia, Peru, Bolivia, Paraguay, Uruguay, Argentina}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{FrenchGuiana, <span style="COLOR: blue">new</span>\[\] { Brazil, Suriname }},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Suriname, <span style="COLOR: blue">new</span>\[\] {Brazil, FrenchGuiana, Guyana}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Guyana, <span style="COLOR: blue">new</span>\[\] {Brazil, Suriname, Venezuala}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Venezuala, <span style="COLOR: blue">new</span>\[\] {Brazil, Guyana, Colombia}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Colombia, <span style="COLOR: blue">new</span> \[\] {Brazil, Venezuala, Peru, Ecuador}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Ecuador, <span style="COLOR: blue">new</span>\[\] {Colombia, Peru}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Peru, <span style="COLOR: blue">new</span>\[\] {Brazil, Colombia, Ecuador, Bolivia, Chile}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Chile, <span style="COLOR: blue">new</span>\[\] {Peru, Bolivia, Argentina}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Bolivia, <span style="COLOR: blue">new</span>\[\] {Chile, Peru, Brazil, Paraguay, Argentina}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Paraguay, <span style="COLOR: blue">new</span>\[\] {Bolivia, Brazil, Argentina}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Argentina, <span style="COLOR: blue">new</span>\[\] {Chile, Bolivia, Paraguay, Brazil, Uruguay}},  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>{Uruguay, <span style="COLOR: blue">new</span>\[\] {Brazil, Argentina}}  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">};</span>

<span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"></span>

<div class="mine">

We can transform this dictionary mapping nodes to neighbours into a list of edge tuples and build the graph:

</div>

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> sagraph = <span style="COLOR: blue">new</span> Graph(13, <span style="COLOR: blue">from</span> x <span style="COLOR: blue">in</span> SA.Keys  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">                            </span><span style="COLOR: blue">from</span> y <span style="COLOR: blue">in</span> SA\[x\]  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">                            </span><span style="COLOR: blue">select</span> <span style="COLOR: #2b91af">Tuple</span>.Create(x, y));</span>

<div class="mine">

Notice that this illustrates both a strength and a weakness of our earlier design decision to make the graph take a list of edges as its input. The graph could just as easily have taken a dictionary mapping nodes to neighbours, and then we wouldn't have to do this transformation in the first place\! It might have been a bad design decision. Such decisions always have to be made in the context of understanding how the user is actually going to use the type.

Anyway, now finding a four-colour solution is easy with our solver:

</div>

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> sasolver = <span style="COLOR: blue">new</span> Solver(sagraph, 4);  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> solution = sasolver.Solve();  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">foreach</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> (<span style="COLOR: blue">var</span> colour <span style="COLOR: blue">in</span> solution )  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span><span style="COLOR: #2b91af">Console</span>.WriteLine(colour);</span>

<div class="mine">

 

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

</div>

</div>

</div>

