# Spot the defect: Bad comparisons, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/27/2011 6:56:00 AM

-----

Did you notice how last time my length comparison on strings was unnecessarily verbose? I could have written it like this:

 

static int ByLength(string x, string y)  
{  
  if (x == null && y == null) return 0:  
  if (x == null) return -1;  
  if (y == null) return 1;  
  return CompareInts(x.Length, y.Length);  
}

static int CompareInts(int x, int y)  
{  
  // Positive if x is larger, negative if y is larger, zero if equal  
  return x - y;  
}

static Comparison\<T\> ThenBy\<T\>(this Comparison\<T\> firstBy, Comparison\<T\> thenBy)  
{  
  return (x,y)=\>  
  {  
    int result = firstBy(x, y);  
    return result \!= 0 ? result : thenBy(x, y);  
  }  
}

Much nicer\! My string length comparison method is greatly simplified, I can compose comparisons easily with the ThenBy extension method, and I can reuse CompareInts as a helper function in other comparisons I might want to write.

What's the defect now?

.

.

.

.

.

.

This one should be easy after last time. The lengths of strings are always positive integers and in practice they never go above a few hundred million; the CLR does not allow you to allocate truly enormous multi-gigabyte strings. But though CompareInts is safe for inputs which are string lengths, **it is not safe in general**. In particular, for the inputs Int32.MinValue and Int32.MaxValue, the difference is 1. Clearly the smallest possible integer is smaller than the largest possible integer, but this method gives the opposite result\! CompareInts should read:

 

static int CompareInts(int x, int y)  
{  
  if (x \> y) return 1;  
  if (x \< y) return -1;  
  return 0;  
}

The moral of the story here is that **a comparison function that doesn't compare something is probably wrong**. Subtraction is not comparison.

