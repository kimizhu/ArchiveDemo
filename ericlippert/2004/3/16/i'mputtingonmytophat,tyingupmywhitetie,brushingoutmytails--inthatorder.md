# I'm Putting On My Top Hat, Tying Up My White Tie, Brushing Out My Tails -- In That Order

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/16/2004 6:33:00 PM

-----

I thought I might blog briefly on an interesting algorithm which can be implemented very elegantly in JScript, but a lot of scripters don't know about. 

Most scripters know about the sort method on the Array prototype -- you give it an array and (optionally) a comparison function, and it sorts the array so that every element is less than or equal to the following element.  However, [as Raymond pointed out a while back](http://weblogs.asp.net/oldnewthing/archive/2003/10/23/55408.aspx "http://weblogs.asp.net/oldnewthing/archive/2003/10/23/55408.aspx"), **the standard sort algorithm only works if your comparison function can impose a total order**.  What if you only care about a **partial order**?  That is, you know that some things must come before others, but there are some items that can sensibly come in any order?  You can't use the built-in sort method for that, and Raymond never did say how to do it in his post. 

Let me give you a concrete example of what I mean -- about twice a year I end up going to some event that requires me to wear my formal evening clothes.  (Who do I think I'm fooling here?  I live in Seattle, the least formally dressed city in America\!  I should say, I go to some event *where I can get away with* wearing formal evening wear, and I'll take every chance I can get to amortize the cost of that tailcoat\!)  Of course, this presents a problem -- what goes on first?  You don't want to put on the vest before the shirt, but it doesn't really matter whether the top hat goes on before or after the gloves.  What we need is a **partial ordering**. 

The sort algorithm that produces partial orders is called **topological sort**, and it is very easy to implement in JScript.  First, let's start with a dependency list that defines what comes before what: 

var deps =  
{  
      tophat      : \[ \],  
      bowtie      : \["shirt"\],  
      socks       : \[ \],  
      pocketwatch : \["vest"\],  
      vest        : \["shirt"\],  
      shirt       : \[ \],  
      shoes       : \["trousers", "socks"\],  
      cufflinks   : \["shirt"\],  
      gloves      : \[ \],  
      tailcoat    : \["vest"\],  
      underpants  : \[ \],  
      trousers    : \["underpants"\]  
};

As you can see, I'm using JScript's object literal and array literal syntaxes to define a data structure where each **top-level member maps to a list of the members that we know must come before it**.  Now all we need is to figure out a partial ordering that meets every one of these conditions.  

Here's what we're going to do: we're going to take every item and examine its list of “dependent children“.  We'll then examine their dependencies, and so on until eventually we find one that has **no** dependencies.  For example, suppose we pick "shoes".  It depends on "trousers", which depends on "underpants", which depends on nothing.  OK, great, so we've reached the bottom -- so what? 

This is very helpful because clearly the ones "at the bottom" are the ones you must do first\!  And of all the items at the bottom, you can do them in any order.  That's the crux of the algorithm: if you then remove the current bottom item from consideration and find any other bottom, and keep doing that until everything is gone, you've got yourself a correct partial ordering.  

We can do this very efficiently -- the key is that if we run **backwards up that descent I just described**, we end up with a good partial ordering: "underpants", "trousers", "shoes".  By diving down until we can go no deeper, we guarantee that we'll never list a later node before one that must come earlier.  We just repeat this process recursively for *every dependency of every node*, and we're done.  

Of course, we'll have to make sure that we don't accidentally hit the same item twice.  Both “bowtie” and “vest” depend on “shirt”.  Once we've hit bottom on “shirt” once we want to never do so again, so every time we've examined any item once we'll mark it as "dead". 

How are we going to run back up the list?  We'll just write a recursive function to do the descent; as we return up the stack we'll stick the current item in the stack frame onto our list. 

It'll make sense when you look at the code, honest. 

function topoSort(dependencies)  
{  
      var dead = \[\];  
      var list = \[\]; 

Step one: everything is alive: 

      for (var dependency in dependencies)  
            dead\[dependency\] = false; 

Step two: visit every dependency on the list -- we don't want to miss any\! 

      for (var dependency in dependencies)  
            visit(dependencies, dependency, list, dead);  
      return list;  
} 

function visit(dependencies, dependency, list, dead)  
{ 

Step three: If we're visiting some place we've already been, then we need go no further. 

      if (dead\[dependency\])  
            return; 

Step four: We're about to explore this entire dependency, so mark it as dead right away, and then recursively explore all this dependency's children, looking for the bottom. 

      dead\[dependency\] = true;  
      for (var child in dependencies\[dependency\])  
            visit(dependencies, dependencies\[dependency\]\[child\], list, dead); 

Step five: As we return back up the stack, we know that all of this dependency's children are taken care of.  Therefore, this is a “bottom“.  Therefore, put it on the list. 

      list.push(dependency);  
} 

If we actually run this thing: 

print(topoSort(deps)); 

We end up with a correct partial order: 

tophat,shirt,bowtie,socks,vest,pocketwatch,underpants,trousers,shoes,cufflinks,gloves,tailcoat 

Pretty neat, eh? Not, I confess, the order in which I'd *normally* dress, but it would work. 

One down side to the algorithm as I've implemented it here is that **dependency lists with cycles are not detected**.  Obviously there can be no partial ordering if **foo** comes before **bar** AND **bar** comes before **foo**, but this algorithm imposes one anyway rather than producing an error.  That's a subject for another entry though.

