# Spot the defect: Bad comparisons, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/20/2011 6:16:00 AM

-----

The mutable List\<T\> class provides an in-place sort method which can take a comparison delegate. It's quite handy to be able to sort a list into order by being able to compare any two elements, but you have to make sure you get it right.

First off, what are the requirements of the comparison delegate? They are [clearly documented](http://msdn.microsoft.com/en-us/library/tfakywbh.aspx): the comparison takes two elements and returns a 32 bit signed integer. If the first element is greater than the second then the integer is greater than zero. If the first element is less than the second then the integer is less than zero. If the first element is equal to the second then the integer is zero.

See if you can figure out why each of these comparisons can give bad results.

Comparison \#1: Putting on my top hat:

 

enum Clothes  
{  
  Hat,  
  Tie,  
  Socks,  
  Pocketwatch,  
  Vest,  
  Shirt,  
  Shoes,  
  Cufflinks,  
  Gloves,  
  Tailcoat,  
  Underpants,  
  Trousers  
}

static int Compare(Clothes x, Clothes y)  
{  
  const int xGreater = 1;  
  const int yGreater = -1;  
  
  // If x has to go on after y then x is the greater  
  // If y has to go on after x then y is the greater  
  // Otherwise, they are equal  
  
  switch (x)  
  {  
  case Clothes.Tie:  
    if (y == Clothes.Shirt) return xGreater;  
    break;  
  case Clothes.Socks:  
    if (y == Clothes.Shoes) return yGreater;  
    break;  
  case Clothes.Pocketwatch:  
    if (y == Clothes.Shirt || y == Clothes.Vest) return xGreater;  
    break;  
  case Clothes.Vest:  
    if (y == Clothes.Shirt) return xGreater;  
    if (y == Clothes.Tailcoat || y == Clothes.Pocketwatch) return yGreater;  
    break;  
  case Clothes.Shirt:  
    if (y == Clothes.Tie || y == Clothes.Pocketwatch ||  
      y == Clothes.Vest || y == Clothes.Cufflinks || y == Clothes.Tailcoat)  
      return yGreater;  
    break;  
  case Clothes.Shoes:  
    if (y == Clothes.Trousers || y == Clothes.Socks || y == Clothes.Underpants)  
      return xGreater;  
  break;  
  case Clothes.Cufflinks:  
    if (y == Clothes.Shirt) return xGreater;  
    break;  
  case Clothes.Tailcoat:  
    if (y == Clothes.Vest || y == Clothes.Shirt) return xGreater;  
    break;  
  case Clothes.Underpants:  
    if (y == Clothes.Trousers || y == Clothes.Shoes) return yGreater;  
    break;  
  case Clothes.Trousers:  
    if (y == Clothes.Underpants) return xGreater;  
    if (y == Clothes.Shoes) return yGreater;  
      break;  
  }  
  return 0;  
}

OK, before you read on, can you figure out what the defect here is? It seems perfectly straightforward: if two things have to be ordered then they are ordered, and if not, then we say we don't care by setting them equal.

Does that work?

.

.

.

.

.

.

.

If you actually try it out on a real list of randomly ordered clothes, you'll find that much of the time this sorts the list into a senseless order, not into an order that preserves nice properties like shoes going on after trousers. Why?

An undocumented but extremely important assumption of the sorting algorithm is that **the comparison function provides a consistent, total order**. Part of providing a total order means that the comparison must preserve the invariant that **things equal to the same are equal to each other**. This idea that if you don't care about the order of two things then you can call them equal is [simply false](http://blogs.msdn.com/b/oldnewthing/archive/2003/10/23/55408.aspx). In our example Tie is equal to Hat and Hat is equal to Shirt, and therefore the sort algorithm is justified in believing that Tie should be equal to Shirt, but it isn't.

Suppose, for example, the first element is "Hat". A sort algorithm is perfectly justified in scanning the entire list, determining that everything is equal to the first element, and concluding that therefore **it must already be sorted**. Clearly a list where every element is equal to the first element is sorted\! The actual implementation of QuickSort in the BCL is quite complex and has several clever tricks in it that help optimize the algorithm for common cases, such as subsets of the list already being in sorted order. If the comparison is not consistent then it is easy to fool those heuristics into doing the wrong thing. And in fact, some sort algorithms will go into infinite loops or crash horribly if you give them an incomplete comparison function.

The right algorithm for sorting a set with a partial order is [topological sort](http://blogs.msdn.com/b/ericlippert/archive/tags/topological+sort/); use the right tool for the job.

Next time: we do the same thing, backwards

