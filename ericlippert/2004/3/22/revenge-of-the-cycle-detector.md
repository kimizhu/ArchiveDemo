<div id="page">

# Revenge of The Cycle Detector

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/22/2004 9:19:00 AM

-----

<div id="content">

<div>

<span> </span>

<span></span>

<span>Mike Schinkel takes even longer to get to the point than I do, and that's saying something\!  Mike tells a long story about [another application of partial order sorts](http://blogs.xtras.net/mikes/PermaLink,guid,57000f55-832e-47e7-9895-2658d4e65d52.aspx "http://blogs.xtras.net/mikes/PermaLink,guid,57000f55-832e-47e7-9895-2658d4e65d52.aspx"), and asks how to modify the partial order sort algorithm so that it has a new property.  Namely, in addition to returning a list ordered such that every item comes after all of its precursors, he wants the list to be ordered so that the set of all the "no precursor" items happen first, and then the set of everything that depends **<span>only</span>** on that first set happens next, and so on. </span>

<span></span>

<span>Why's that?  Because then each set can be done entirely **<span>in parallel</span>** -- all the items in each set don't depend on each other, and so if you have multiple processors, you can assign a different processor to each item in the current set and be guaranteed that you'll still do everything in a good order.  That is, do all of the first set in parallel, then all of the second set in parallel, and so on.  (This isn't the **<span>best possible</span>** processor scheduling algorithm for this class of problems, but it's pretty decent.  Exercise for the reader: improve it\!) </span>

<span></span>

<span>The trick here is to define the "height" of a node as the length of the **<span>longest</span>** chain of child dependencies you can make that include it.  Clearly all of the "height one" nodes -- the ones with only themselves on the chain of child dependencies because they have no children -- can be done in parallel.  All of the "height two" nodes -- that only have "height one" children -- can be done in parallel once the height one nodes are done, and so on. </span>

<span></span>

<span>This gives us a straightforward way to compute the height -- the height is one if you have no children, otherwise it is one plus the height of your tallest child. </span>

<span></span>

**<span>var unknown = 0;  
</span><span>var undead = -1;  
</span>**<span>function topoSort(dependencies)  
</span><span>{  
</span><span>**<span>  var height = \[\];  
</span>**</span><span>  var list = \[\];  
</span><span>  for (var dependency in dependencies)  
</span><span><span>  <span>  </span></span>height\[dependency\] = unknown;   
</span><span><span>  </span>for (var dependency in dependencies)  
</span><span><span>  <span>  </span></span>visit(dependencies, dependency, list, height);  
</span><span><span>  </span>return list;  
</span><span>} </span>

<span></span>

<span>function visit(dependencies, dependency, list, height)  
</span><span>{   
</span><span><span>  </span>if (height\[dependency\] == undead)  
</span><span><span>  <span>  </span></span>throw "Hey, you've got a cycle in dependency " + dependency;  
</span><span><span>  </span>if (height\[dependency\] \!= unknown)  
</span><span><span><span>  </span>  </span>return height\[dependency\];  
</span><span><span>  </span>**var max = 0;  
**</span><span><span>  </span>height\[dependency\] = undead;  
</span><span><span>  </span>for (var child in dependencies\[dependency\])  
</span><span><span>  </span>{  
</span><span><span>  <span>  </span></span>var childheight = visit(dependencies, dependencies\[dependency\]\[child\], list, height);   
</span><span><span>  <span>  </span></span>**if (childheight \> max)  
**</span><span><span>  <span>  <span>  </span></span></span>**max = childheight;  
**</span><span><span>  </span>}  
</span><span><span>  </span>**height\[dependency\] = max + 1;  
**</span><span><span>  </span>**if (typeof(list\[max + 1\]) == "undefined")  
**</span><span><span>  <span>  </span></span>**list\[max + 1\] = \[\];  
**</span><span><span>  </span>**list\[max + 1\].push(dependency);  
**</span><span><span>  </span>**return max + 1;  
**</span><span>} </span>

<span></span>

<span>print(topoSort(deps).join("n")); </span>

<span></span>

<span>Which produces </span>

<span></span>

<span>tophat,shirt,socks,underpants,gloves  
</span><span>bowtie,vest,trousers,cufflinks  
</span><span>pocketwatch,shoes,tailcoat </span>

<span></span>

<span>Which is still not the order in which I'd normally dress, but again, it would work.</span>

</div>

</div>

</div>

