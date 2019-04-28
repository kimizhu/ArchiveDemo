# Bad Hungarian

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 4:37:00 PM

-----

 

One more word about Hungarian Notation and then I'll let it drop, honest.  (Maybe.)  
   
If you're going to uglify your code with Hungarian Notation, please, at least do it right.  The whole point is to make the code easier to read and reason about, and that means ensuring that the invariants expressed by the Hungarian prefixes and suffixes are actually invariant.  
   
Here's some code I found once in the "diagram save" code of a Visio-like tool:  
   
long lCountOfConnectors = srpConnectors-\>Count();  
while( --lCountOfConnectors)  
{  
      // \[Omitted: get the next connector\]  
      // \[Omitted: save the connector to a stream\]  
}  
   
OK, first of all, that should be cConnectors.  But that's just a trivial question of what lexical convention we use.  There's a far more serious problem here. The *number of connectors is not decreasing as we iterate the loop, so* cConnectors *should not be decreasing*.   
   
**Hungarian makes it easier to reason about code, but only when you make sure that the algorithm semantics and the Hungarian semantics match**.  Seriously, when I first read this code I naturally assumed that it was removing connectors from the collection for some reason, and therefore decreasing the count so that the count would continue to match reality.  But in fact it was just using the count as an index, which is wrong.  The code should read something like:  
   
long cConnectors = srpConnectors-\>Count();  
for(long iConnector = 0 ; iConnector \< cConnectors ; ++iConnector)  
{  
      // \[...\]  
}  
   
In other words, **the name of a variable should reflect its meaning throughout its lifetime, not merely its initialization.** ****

