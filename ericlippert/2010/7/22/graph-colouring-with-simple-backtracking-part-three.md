<div id="page">

# Graph Colouring with Simple Backtracking, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/22/2010 6:54:00 AM

-----

<div id="content">

<div class="mine">

OK, we've got our basic data structures in place.

Graph colouring is a very well-studied problem. It's known to be NP-complete for arbitrary graphs, so (assuming that P\!=NP) we're not going to find an always-fast algorithm for colouring an arbitrary graph. However, for typical graphs that we encounter in the wild, the following simple algorithm is pretty good. Start by saying that every node can be every possible colour. Then:

1\) Do you have a single possible colouring for *every* node in the graph? Success\!  
2\) Do you have *any* node that cannot possibly have any colour? Failure\!  
3\) Otherwise, you must have a node that still has at least two possible colours.  
4\) Some of the nodes might have a single possible colour and a neighbour that has that possible colour. Remove the possibility from the neighbour.  
5\) That might have caused some of the neighbours to themselves be reduced to having just one possible colour. If so, remove *those* colours from the possibilities of the neighbours of those nodes.  
6\) And so on; keep on reducing the possibilities until you can do so no longer.  
7\) Did you eliminate a possibility? Then maybe we've got a solution, or maybe we're now busted. Go back to step 1.  
8) Otherwise, you eliminated no possibilities and there was a node that had two or more possible colours. Hypothesize that the node is one of those colours.   
9\) Recurse; If the recursion produced success, then you're done. If the recursion failed, then your hypothesis was faulty.  
10\) If there is an untried hypothesis for the node, go back to step eight and try a different hypothesis.  
11\) Otherwise none of the possible values for one of the undetermined nodes are good; fail.

Notice that this is essentially a "backtracking" algorithm. We make a hypothesis, and if that hypothesis pans out, great, we're done. If not, we backtrack and make a different hypothesis. If none of those hypotheses work out then either the problem is not solvable (for example, we are trying to colour South America with three colours) or a *previous* hypothesis was wrong, so we backtrack even further. We keep doing that until we've tried all possible combinations.

That last bit is what makes this potentially exponential in time. In realistic graphs, the "reduction" step quickly eliminates so many possibilities that we don't have to explore nearly the entire space of possibilities before either finding a solution or concluding that there is none.

How are we going to do this? Remember, my goals here are to make the program easier to reason about by keeping things immutable where possible, but not so immutable that doing so produces bad performance or weird code.

What we'll do is we'll make a class called a "Solver" which represents a bunch of possible colours for each node. The solver is immutable; when we must make a hypothesis or a deduction that changes what the possible colours are for a node, we'll make a new solver rather than mutating the old one. Let's make a solver that attempts to give a solution in n colours for a particular graph:

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">internal</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">sealed</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">class</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Solver  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
  </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">enum</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> { Solved, Unsolved, Busted }  
  </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private readonly</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Graph</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> graph;  
  </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private readonly</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\[\] possibilities;  
  </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private readonly</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> colours;  
  </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">public</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> Solver(</span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Graph</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> graph, </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> colours)  
  </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.colours = colours;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.graph = graph;  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">    this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities = </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">new</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\[graph.Size\];  
    </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> b = </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Empty;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">for</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> i = 0; i \< colours; ++i)  
        </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">b = b.Add(i);  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">for</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> p = 0; p \< </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities.Length; ++p)  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities\[p\] = b;  
 </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}</span>

Pretty straightforward so far. The class is internal and sealed; we don't expect anyone to derive from this or use it outside of this assembly. Later on we're going to implement the first few lines of the algorithm sketch by asking "are we solved, are we busted, or do we still have more work to do?" so I'll represent that by an enum. The solver keeps a graph, the number of colours it is going to try to use to colour the graph, and a bitset of possibilities for each node. The set of possibilities for each node is an array; as we'll see later, using a mutable data structure for this task has pros and cons.

We initialize the array of possibilities to have one set per node, and say that each node can be every one of the possible colours.

Again, there's no error handling in here. Were this a production-grade library I'd have logic that verified that the number of colours was between 1 and 32.

Later on I'm going to need a private constructor that can set all of these fields directly, so let's just build that now:

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">    private</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> Solver(</span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Graph</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> graph, </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> colours, </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\[\] possibilities)  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.graph = graph;  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.colours = colours;  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities = possibilities;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}</span>

We'll need to be able to make a new solver with one of the node's possible colours reduced to a single possibility.

<span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"></span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">    // Make a new solver with one of the colours known.  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">    public</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Solver</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> SetColour(</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> node, </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> colour)  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">    {  
</span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">        BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\[\] newPossibilities = (</span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\[\])</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities.Clone();  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">        newPossibilities\[node\] = </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">BitSet</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Empty.Add(colour);  
</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">        return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">new</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Solver</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">(</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.graph, </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.colours, newPossibilities);  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}</span></span>

Here we now see the pros and cons of using an array of bitsets. The bitsets were designed to be small and lightweight value types; they're just a wrapper on an int. Copying a bitset is trivial; we just copy the int. However, copying the *array* and then mutating it is more expensive; we duplicate the whole thing. If these arrays were going to be huge then this could get very inefficient; every time we'd make a new solver out of an old one we'd have huge amounts of duplication between the two; the difference between the two would in some cases be a single bit, which seems wasteful. If that were the case then instead of using an array of bitsets, I'd use some sort of immutable collection of bitsets that enabled me to re-use most of the state upon making an edit. That would add a lot of complexity, and frankly, copying small arrays is really fast and not that big. So, with the assumption that the graphs we are colouring are small, we'll stick with the array solution.

Note that our choice to make the sets immutable is paying off. Suppose we'd chosen a mutable set. If we mutated the set by removing all the items but one, we'd have to remember to put them back if this hypothesis doesn't pan out\!<span></span><span></span>

 I've made this method public. It seems reasonable that the caller of the solver might want to impose certain restrictions of their own, like "I already know that I want Brazil to be green." And as we'll see in a couple more posts, this will come in handy.

Anyway, getting back to the algorithm. The first thing we're going to need to know is "are we done?"

<span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> Status  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">get  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (possibilities.Any(p =\> \!p.Any()))  
            </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Busted;  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (possibilities.All(p =\> p.Count() == 1))  
            </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Solved;  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Unsolved;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}  
}</span></span>

Do not be fooled by its commonplace appearance; there are several interesting design choices here. First, we're saying that if there is any node that has no possibilities, then we're busted. The way I've chosen to do so is *clever*, and that's maybe *bad*. I don't like clever code, I like code that is easy to read, and this is getting a bit cute. It seems likely that the nested use of "Any" to determine "is there any member of this collection of possibilities that does not have any possibility?" is hard to read. I'm trying to eschew loops here, to make the code more declarative and less imperative, but this might have gone too far.

Second, we're saying that if we're not busted, then maybe we have a solution: exactly one colour per node. Notice that I am counting each possibility set to determine if there is exactly one. That could do up to 32 bit operations, when we could stop after two.

Notice also that we are potentially iterating over the entire array of possibilities twice, looking for what is essentially the same information. I could have written this code like this:

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">private</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> Status  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">get  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
        <span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">foreach</span></span>(var p <span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">in </span></span>possibilities)  
        {  
            <span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span>primitiveCount = p.ZeroOneOrMany();  
            if (primitiveCount == 0) <span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span>.Busted;  
            if (primitiveCount \> 1) <span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span>.Unsolved;  
</span><span></span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">        }  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Solved;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}  
}</span></span>

And then written an extension method ZeroOneOrMany() that counts the first *two* items of a sequence only.

Here I'm choosing to make the code be more declarative and read more like the algorithm at the cost of a little efficiency. I'm not taking on the burden of writing an unnecessary special-purpose counting algorithm. I know that counting a bitset is going to be pretty cheap; I can afford to have a few extra loops to calculate counts I don't actually need.

Notice also that I am not caching the result; it is entirely possible that this solver ask itself multiple times what its status is, and we recompute it every time. We have a tradeoff here of extra space and redundancy in exchange for better speed. But do we actually have a scenario where one solver is going to be asked multiple times what its status is? I don't think we do, so again, YAGNI. Let's not add extra complexity that we're not going to use, not without a clear benefit.

The basic operation of this algorithm has two phases. First we made all the deductions we can trivially do so without making any hypothesis, and then make a hypothesis. How are we going to do that first part? Again, there is a tension between an easy solution that involves mutating an array and a more pure but somewhat more complex solution that creates an immutable map from nodes to possible colours and then gradually edits the map by producing new maps from it. We'll encapsulate the mutation solution by making it an implementation detail of the reduction method. The idea here is that the reduction method returns either null, if no reductions could be performed, or a new immutable solver with all the reductions performed.

<span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: blue;">private</span><span style="color: #000000;"> </span><span style="color: #2b91af;">Solver</span><span style="color: #000000;"> Reduce()  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;">{  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: #2b91af;">BitSet</span><span style="color: #000000;">\[\] newPossibilities = (</span><span style="color: #2b91af;">BitSet</span><span style="color: #000000;">\[\])</span><span style="color: blue;">this</span><span style="color: #000000;">.possibilities.Clone();  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: green;">// The query answers the question "What colour possibilities should I remove?"  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: blue;">var</span><span style="color: #000000;"> reductions =  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">from</span><span style="color: #000000;"> node </span><span style="color: blue;">in</span><span style="color: #000000;"> </span><span style="color: #2b91af;">Enumerable</span><span style="color: #000000;">.Range(0, newPossibilities.Length)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">where</span><span style="color: #000000;"> newPossibilities\[node\].Count() == 1  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">let</span><span style="color: #000000;"> colour = newPossibilities\[node\].Single()  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">from</span><span style="color: #000000;"> neighbour </span><span style="color: blue;">in</span><span style="color: #000000;"> graph.Neighbours(node)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">where</span><span style="color: #000000;"> newPossibilities\[neighbour\].Contains(colour)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">select</span><span style="color: #000000;"> </span><span style="color: blue;">new</span><span style="color: #000000;"> { neighbour, colour };  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: blue;">bool</span><span style="color: #000000;"> progress = </span><span style="color: blue;">false</span><span style="color: #000000;">;  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: blue;">while</span><span style="color: #000000;">(</span><span style="color: blue;">true</span><span style="color: #000000;">)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;">    </span>{  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">var</span><span style="color: #000000;"> list = reductions.ToList();  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">if</span><span style="color: #000000;"> (list.Count == 0)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">            </span></span><span style="color: blue;">break</span><span style="color: #000000;">;  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;">        </span>progress = </span><span style="color: blue;">true</span><span style="color: #000000;">;  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: blue;">foreach</span><span style="color: #000000;"> (</span><span style="color: blue;">var</span><span style="color: #000000;"> reduction </span><span style="color: blue;">in</span><span style="color: #000000;"> list)  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;">            </span>newPossibilities\[reduction.neighbour\] = newPossibilities\[reduction.neighbour\].Remove(reduction.colour);  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: green;">// Doing so might have created a new node that has a single possibility,  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: green;">// which we can then use to make further reductions. Keep looping until  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">        </span></span><span style="color: green;">// there are no more reductions to be made.  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;"><span style="mso-spacerun: yes;">    </span>}  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="mso-spacerun: yes;"><span style="color: #000000;">    </span></span><span style="color: blue;">return</span><span style="color: #000000;"> progress ? </span><span style="color: blue;">new</span><span style="color: #000000;"> </span><span style="color: #2b91af;">Solver</span><span style="color: #000000;">(graph, colours, newPossibilities) : </span><span style="color: blue;">null</span><span style="color: #000000;">;  
</span></span><span style="font-family: Consolas; font-size: 9.5pt;"><span style="color: #000000;">}</span></span> <span></span><span></span>

<span></span><span></span>

<span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt"><span style="font-family: Consolas; color: #0000ff; font-size: x-small;"><span style="font-family: Consolas; color: #0000ff; font-size: x-small;"><span style="font-family: Consolas; color: #0000ff; font-size: x-small;"><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas; font-size: x-small;"></span></span></span></span></span></span> A few things to note here. 

I want to avoid any situation where we are mutating an object while iterating over it. We could in fact get away with that in this case; we could run the query and mutate the array while the query was running. But that's really bad form, and would make it difficult to refactor this code, should we later decide to use a different mutable collection, or use an immutable collection for the possibilities. Instead I run the query once, cache the results in a list, and then iterate over the list.

Note that again I'm trying to use queries to represent operations that ask questions, and loops to represent operations that induce mutations. And note that again I am using Count() rather than writing a custom method that checks "does this sequence have exactly one element", which could then bail out if there were two or more.

On the other hand, I'm using List.Count rather than the Any() extension method because I know that List.Count is actually cheaper than Any(). Normally the Any() extension method is cheaper than the Count() extension method because counting a sequence might involve iterating the whole thing, whereas the Any predicate simply calls MoveNext once.

Notice that we could take an early out along the way here. We could detect whether or not we were in a "busted" configuration and return that result, rather than continuing to do calculations. The question is whether the cost of doing the test every time is worth the benefit of taking the early out. I'm going to assume not.

I'm not normally a huge fan of using null references to signal conditions, but here it seems justified. What's the alternative? Return a tuple of (bool, Solver) that says whether progress was made? That seems weird. This question is much more interesting in our next chunk of code.

Which brings us to the solver algorithm itself:

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">public</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">IEnumerable</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\<</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\> Solve()  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
  </span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">// Base case: we are already solved or busted.  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">var</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> status = </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Status;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (status == </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Solved)  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities.Select(x =\> x.Single());  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (status == </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Busted)  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">null</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">// Easy inductive case: do simple reductions and then solve again.  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">var</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> reduced = Reduce();  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (reduced \!= </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">null</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">)  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> reduced.Solve();  
    </span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">// Difficult inductive case: there were no simple reductions.  
    </span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">// Make a hypothesis about the colouring of a node and see if  
    </span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">// that introduces a contradiction or a solution.  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> node = <span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.</span>possibilities.FirstIndex(p =\> p.Count() \> 1);  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">var</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> solutions =  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">from</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> colour </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">in</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> <span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.</span>possibilities\[node\]  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">let</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> solution = </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">this</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.SetColour(node, colour).Solve()  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">where</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> solution \!= </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">null  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">select</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> solution;  
    </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> solutions.FirstOrDefault();  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">}</span>

 

I've used an extension method here that gets you the index of the first item that matches a predicate:

<span style="font-family: Consolas; color: blue; font-size: 10pt;">public</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">static</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">int</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> FirstIndex\<T\>(</span><span style="font-family: Consolas; color: blue; font-size: 10pt;">this</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> </span><span style="font-family: Consolas; color: #2b91af; font-size: 10pt;">IEnumerable</span><span style="font-family: Consolas; color: black; font-size: 10pt;">\<T\> items, </span><span style="font-family: Consolas; color: #2b91af; font-size: 10pt;">Func</span><span style="font-family: Consolas; color: black; font-size: 10pt;">\<T, </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">bool</span><span style="font-family: Consolas; color: black; font-size: 10pt;">\> predicate)  
</span><span style="font-family: Consolas; color: black; font-size: 10pt;">{  
  </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">int</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> i = 0;  
  </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">foreach</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> (T item </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">in</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> items)  
  </span><span style="font-family: Consolas; color: black; font-size: 10pt;">{  
    </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">if</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> (predicate(item))  
      </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">return</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> i;  
    </span><span style="font-family: Consolas; color: black; font-size: 10pt;">i++;  
  </span><span style="font-family: Consolas; color: black; font-size: 10pt;">}  
  </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">throw</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> </span><span style="font-family: Consolas; color: blue; font-size: 10pt;">new</span><span style="font-family: Consolas; color: black; font-size: 10pt;"> </span><span style="font-family: Consolas; color: #2b91af; font-size: 10pt;">Exception</span><span style="font-family: Consolas; color: black; font-size: 10pt;">();  
</span><span style="font-family: Consolas; color: black; font-size: 10pt;">}</span>

Note that it requires that at least one item match the predicate\! This might not be a good general-purpose design but it meets my needs. A more general-purpose design might return a nullable int instead.

Anyway, nNow we have the public interface to the solver; it returns a sequence of colourings for each node, if there is one, or null, if there is no such sequence. This illustrates an important principle about sequences: the null sequence is not the empty sequence. The null sequence is "there is no such sequence", not "there is such a sequence and it is empty".

I'm not sure that a sequence is really the right thing to return here though. What we want to express is that there is a mapping from node number to colour number. But how would the user say "what's the colour of node number twelve?"  You can't index into the sequence\! The caller is almost certainly going to call ToArray or ToList  or ToLookup on this thing, and maybe it would be sensible for us to do that for them. On the other hand, it's the caller that knows whether they want an array or a list or a dictionary or what, so maybe we shouldn't make that decision for them. I'll leave it like this for now.

I note that for the first recursive case we don't actually need to use recursion. We could have written the first part this like:

 

<span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">public</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">IEnumerable</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\<</span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">int</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">\> Solve()  
</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">{  
    var current = this;  
    while(true)  
    {  
</span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">var</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> status = current</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Status;  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (status == </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Solved)  
            </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> current</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.possibilities.Select(x =\> x.Single());  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (status == </span><span style="FONT-FAMILY: Consolas; COLOR: #2b91af; FONT-SIZE: 10pt">Result</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">.Busted)  
            </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">return</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">null</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">;  
</span><span style="FONT-FAMILY: Consolas; COLOR: green; FONT-SIZE: 10pt">        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">var</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> reduced = Reduce();  
        </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">if</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"> (reduced == </span><span style="FONT-FAMILY: Consolas; COLOR: blue; FONT-SIZE: 10pt">null</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">)  
            break;</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">  
        </span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt">current = reduced;  
    }</span><span style="FONT-FAMILY: Consolas; COLOR: black; FONT-SIZE: 10pt"></span>

 

Which has the nice property of not recursing unnecessarily. But it is harder to reason about (if you reason about recursion easily; not everyone does) and reads less like our English description of the algorithm.

The major recursive step has an interesting use of a query expression. We produce a query that will create a sequence of all solutions for all possible hypotheses for the first node that can have more than one possible colouring. But all we care about is "is there any non-null solution or not?" FirstOrDefault will fetch the first solution, if there is one, and return null otherwise. We're taking advantage of the fact that queries are lazy; we don't calculate all the possible solutions unless we actually iterate over the whole query.

Note that if we needed to we could modify this so that we asked for the first two solutions, if there are two. That way we could tell whether the solution is *unique* or not\! But right now we are looking only for any solution.

Next time: let's take this algorithm out for a test drive. South America, here we come.

(Eric is out camping in the beautiful Pacific Northwest; this posting was prerecorded.) 

 

</div>

</div>

</div>

