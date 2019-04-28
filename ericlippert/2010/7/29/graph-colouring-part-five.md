<div id="page">

# Graph Colouring, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/29/2010 6:40:00 AM

-----

<div id="content">

<div class="mine">

I said last time that I was interested in finding colourings for graphs that have lots of fully connected subgraphs, aka "cliques". For instance, I'd like to find a four-colouring for this sixteen-node graph:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/8032.FourGraph1.png)

Yuck. What a mess.

What this graph is doing a bad job of conveying is that there are twelve fully connected subsets. {0, 1, 2, 3} forms a clique. So does {0, 1, 4, 5}. And so does {0, 4, 8, 12}.

It would be great if I had a better way to display full connectedness. How about this: I'll just draw a dotted ellipse around a fully connected subset, and you can then mentally fill in all the edges:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/1803.FourGraph2.png)

Hmm. That might not actually be better.

Anyway, you get the idea. What would be really nice is if I had a way to cleanly represent this in code. What I want to do is somehow take the concept of a sequence of fully connected subsets:

{0, 1, 2, 3}, {4, 5, 6, 7}, {8, 9, 10, 11}, {12, 13, 14, 15},  
{0, 4, 8, 12}, {1, 5, 9, 13}, {2, 6, 10, 14}, {3, 7, 11, 15},  
{0, 1, 4, 5}, {2, 3, 6, 7}, {8, 9, 12, 13}, {10, 11, 14, 15}

and turn that into a list of edges: {0, 1}, {0, 2}, {0, 3}, {1, 0}, {1, 2}, ...

If I end up with some of the edges on the list twice, who cares? I've implemented my graph as a Boolean array; no problem if I set one of the entries to true twice.

This is easily done with LINQ:

</div>

<span style="font-family: Consolas; color: blue; font-size: 9.5pt;">public</span><span style="font-family: Consolas; font-size: 9.5pt;"> <span style="color: blue;">static</span> <span style="color: #2b91af;">IEnumerable</span>\<<span style="color: #2b91af;">Tuple</span>\<<span style="color: blue;">int</span>, <span style="color: blue;">int</span>\>\> CliquesToEdges(<span style="color: #2b91af;">IEnumerable</span>\<<span style="color: #2b91af;">IEnumerable</span>\<<span style="color: blue;">int</span>\>\> cliques)  
</span><span style="font-family: Consolas; font-size: 9.5pt;">{  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">  </span><span style="color: blue;">return  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">from</span> clique <span style="color: blue;">in</span> cliques  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">from</span> item1 <span style="color: blue;">in</span> clique   
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">from</span> item2 <span style="color: blue;">in</span> clique   
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">where</span> item1 \!= item2  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">select</span> <span style="color: #2b91af;">Tuple</span>.Create(item1, item2);  
</span><span style="font-family: Consolas; font-size: 9.5pt;">}</span>

<div class="mine">

 

 Now, I suppose we could just make an array of arrays with all the numbers I listed above. But notice somthing about the pattern: 

 In that first line, we start with {0, 1, 2, 3}. Then we add four to each: {4, 5, 6, 7}. Then we add eight: {8, 9, 10, 11}. Then we add twelve: {12, 13, 14, 15}.

In the second line, we start with {0, 4, 8, 12}. Then we add one: {1, 5, 9, 13}. Then we add two. Then three.

In the third line, we start with  {0, 1, 4, 5}. Then we add two, eight and ten.

In short, we could write a generator for the subsets if we wanted to, by noting down the patterns:

 

</div>

<span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"> offsets = <span style="color: blue;">new</span>\[,\] {  
</span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*rows\*/</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span>{<span style="color: blue;">new</span>\[\] {0, 4, 8, 12}, <span style="color: blue;">new</span>\[\] {0, 1, 2, 3 }},  
</span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*columns\*/</span><span style="font-family: Consolas; font-size: 9.5pt;"> {<span style="color: blue;">new</span>\[\] {0, 1, 2, 3},<span style="mso-spacerun: yes;">  </span><span style="color: blue;">new</span>\[\] {0, 4, 8, 12}},  
</span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*squares \*/</span><span style="font-family: Consolas; font-size: 9.5pt;">{<span style="color: blue;">new</span>\[\] {0, 2, 8, 10}, <span style="color: blue;">new</span>\[\] {0, 1, 4, 5}}};  
</span><span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"> cliques =  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">from</span> r <span style="color: blue;">in</span> Enumerable.Range(0, 3)  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">from</span> i <span style="color: blue;">in</span> offsets\[r, 0\]  
</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;">    </span><span style="color: blue;">select</span> (<span style="color: blue;">from</span> j <span style="color: blue;">in</span> offsets\[r, 1\] <span style="color: blue;">select</span> i + j);  
</span><span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"> edges = CliquesToEdges(cliques);  
</span><span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"> graph = <span style="color: blue;">new</span> Graph(16, edges);  
</span><span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"> solver = <span style="color: blue;">new</span> Solver(graph, 4);</span>

<div class="mine">

And if we solve that  we get the colours 0, 1, 2, 3, 2, 3, 0, 1, 1, 0, 3, 2, 3, 2, 1, 0 for the 16 nodes:

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/1715.FourGraph3.png)

And sure enough, every fully-connected subset with an ellipse around it has no two nodes of the same colour. Neat\!

But let's not stop there\! What if instead of sixteen nodes we had... eighty-one\! And what if instead of twelve fully-connected subsets, we had... twenty-seven\!

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/0535.NineGraph.png)

And what if we knew ahead of time what some, but not all of the colourings were?

First off, we can modify our subset generator easily enough:

</div>

<span style="font-family: Consolas; color: blue; font-size: 9.5pt;">var</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"> offsets = </span><span style="color: blue;">new</span><span style="color: #000000;">\[,\] {  
</span></span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*rows\*/</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;">   </span>{</span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 9, 18, 27, 36, 45, 54, 63, 72}, </span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 1, 2, 3, 4, 5, 6, 7, 8}},  
</span></span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*columns\*/</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;">{</span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 1, 2, 3, 4, 5, 6, 7, 8},<span style="mso-spacerun: yes;"> </span></span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 9, 18, 27, 36, 45, 54, 63, 72}},  
</span></span><span style="font-family: Consolas; color: green; font-size: 9.5pt;">/\*boxes \*/</span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;"> </span>{</span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 3, 6, 27, 30, 33, 54, 57, 60},<span style="mso-spacerun: yes;"> </span></span><span style="color: blue;">new</span><span style="color: #000000;">\[\] {0, 1, 2, 9, 10, 11, 18, 19, 20}}};  
</span></span> 

<div class="mine">

<span style="font-size: x-small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"></span></span></span>

<span style="font-size: x-small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"></span></span></span>

The generation of the cliques and edges is the same. Let's make a solver for that graph and then restrict some of the colour choices ahead of time:

</div>

 <span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">Graph graph = <span style="COLOR: blue">new</span> Graph(81, edges);  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> solver = <span style="COLOR: blue">new</span> Solver(graph, 9);  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">string</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> puzzle =  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"<span style="mso-spacerun: yes">  </span>8 274<span style="mso-spacerun: yes">  </span>"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"<span style="mso-spacerun: yes">         </span>"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">" 6 38<span style="mso-spacerun: yes">  </span>52"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"<span style="mso-spacerun: yes">      </span>32 "</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"1<span style="mso-spacerun: yes">   </span>7<span style="mso-spacerun: yes">   </span>4"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">" 92<span style="mso-spacerun: yes">      </span>"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"78<span style="mso-spacerun: yes">  </span>62 1 "</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"<span style="mso-spacerun: yes">         </span>"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> +  
</span><span style="FONT-FAMILY: Consolas; COLOR: #a31515; FONT-SIZE: 9.5pt">"<span style="mso-spacerun: yes">  </span>384 5<span style="mso-spacerun: yes">  </span>"</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">;</span>

<span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"></span>

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">int</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> node = -1;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">foreach</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> (<span style="COLOR: blue">char</span> c <span style="COLOR: blue">in</span> puzzle)  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">{  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>++node;  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span><span style="COLOR: blue">if</span> (c == <span style="COLOR: #a31515">' '</span>) <span style="COLOR: blue">continue</span>;  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>solver = solver.SetColour(node, c - <span style="COLOR: #a31515">'1'</span>);  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">}  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 9.5pt">var</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"> result = solver.Solve();  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="COLOR: blue">int</span> i = 0;  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="COLOR: blue">foreach</span> (<span style="COLOR: blue">var</span> r <span style="COLOR: blue">in</span> result)  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">{  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span><span style="COLOR: #2b91af">Console</span>.Write(r + 1);  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes"> </span><span style="mso-spacerun: yes">   </span><span style="COLOR: blue">if</span> (i % 9 == 8)  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">        </span><span style="COLOR: #2b91af">Console</span>.WriteLine();  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt"><span style="mso-spacerun: yes">    </span>++i;  
</span><span style="FONT-FAMILY: Consolas; FONT-SIZE: 9.5pt">}</span>

<div class="mine">

And we get the output 

358127469  
279654831  
461389752  
847916325  
135278694  
692435187  
784562913  
516793248  
923841576

Holy goodness\! There I was, minding my own business, trying to solve problems in graph theory and I *accidentally* made a Sudoku puzzle solver\! Isn't it funny how life turns out sometimes? But *that's just how awesome LINQ is. *

And with that surprising result, I'm off to my ancestral homeland to spend a couple weeks visiting family and friends. We'll pick up with more Fabulous Adventures in September.

 

</div>

</div>

</div>

