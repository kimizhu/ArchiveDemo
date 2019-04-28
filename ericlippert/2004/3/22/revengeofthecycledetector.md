# Revenge of The Cycle Detector

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/22/2004 9:19:00 AM

-----

 

Mike Schinkel takes even longer to get to the point than I do, and that's saying something\!  Mike tells a long story about [another application of partial order sorts](http://blogs.xtras.net/mikes/PermaLink,guid,57000f55-832e-47e7-9895-2658d4e65d52.aspx "http://blogs.xtras.net/mikes/PermaLink,guid,57000f55-832e-47e7-9895-2658d4e65d52.aspx"), and asks how to modify the partial order sort algorithm so that it has a new property.  Namely, in addition to returning a list ordered such that every item comes after all of its precursors, he wants the list to be ordered so that the set of all the "no precursor" items happen first, and then the set of everything that depends **only** on that first set happens next, and so on. 

Why's that?  Because then each set can be done entirely **in parallel** -- all the items in each set don't depend on each other, and so if you have multiple processors, you can assign a different processor to each item in the current set and be guaranteed that you'll still do everything in a good order.  That is, do all of the first set in parallel, then all of the second set in parallel, and so on.  (This isn't the **best possible** processor scheduling algorithm for this class of problems, but it's pretty decent.  Exercise for the reader: improve it\!) 

The trick here is to define the "height" of a node as the length of the **longest** chain of child dependencies you can make that include it.  Clearly all of the "height one" nodes -- the ones with only themselves on the chain of child dependencies because they have no children -- can be done in parallel.  All of the "height two" nodes -- that only have "height one" children -- can be done in parallel once the height one nodes are done, and so on. 

This gives us a straightforward way to compute the height -- the height is one if you have no children, otherwise it is one plus the height of your tallest child. 

**var unknown = 0;  
var undead = -1;  
**function topoSort(dependencies)  
{  
**  var height = \[\];  
**  var list = \[\];  
  for (var dependency in dependencies)  
    height\[dependency\] = unknown;   
  for (var dependency in dependencies)  
    visit(dependencies, dependency, list, height);  
  return list;  
} 

function visit(dependencies, dependency, list, height)  
{   
  if (height\[dependency\] == undead)  
    throw "Hey, you've got a cycle in dependency " + dependency;  
  if (height\[dependency\] \!= unknown)  
    return height\[dependency\];  
  **var max = 0;  
**  height\[dependency\] = undead;  
  for (var child in dependencies\[dependency\])  
  {  
    var childheight = visit(dependencies, dependencies\[dependency\]\[child\], list, height);   
    **if (childheight \> max)  
**      **max = childheight;  
**  }  
  **height\[dependency\] = max + 1;  
**  **if (typeof(list\[max + 1\]) == "undefined")  
**    **list\[max + 1\] = \[\];  
**  **list\[max + 1\].push(dependency);  
**  **return max + 1;  
**} 

print(topoSort(deps).join("n")); 

Which produces 

tophat,shirt,socks,underpants,gloves  
bowtie,vest,trousers,cufflinks  
pocketwatch,shoes,tailcoat 

Which is still not the order in which I'd normally dress, but again, it would work.

