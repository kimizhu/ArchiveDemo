# Spot the defect: Bad comparisons, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/24/2011 6:43:00 AM

-----

Suppose I want to sort a bunch of strings into order first by length, and then, once they are sorted by length, sort each group that is the same length by some other comparison. We can easily build such a device with higher-order programming:

 

static Comparison\<string\> FirstByLength(Comparison\<string\> thenBy)  
{  
  return (string x, string y) =\>  
  {  
    // Null strings are sorted before zero-length strings; remember, we need to provide a total ordering.  
    if (x == null && y == null)  
      return 0;  
    if (x == null)  
      return -1;  
    if (y == null)  
      return 1;  
    if (x.Length \> y.Length)  
      return 1;  
    if (x.Length \< y.Length)  
      return -1;  
    // They are the same length; sort on some other criterion.  
    return thenBy(x, y);  
  };  
}

Super. This idea of composing new comparison functions out of old ones is pretty neat. We can even built a reversal device:

 

static Comparison\<string\> Reverse(Comparison\<string\> comparison)  
{  
  return (string x, string y) =\> -comparison(x, y);  
}

Something is subtly wrong in at least one of these comparison functions. Where's the defect?

.

.

.

.

.

.

.

.

Let's restate that contract again. The comparison function returns a negative integer if the first argument is smaller than the second, a positive integer if the first is greater than the second, and zero if they are equal. Any negative integer will do, and in particular, Int32.MinValue is a negative integer. Suppose we have a bizarre comparison function that returns Int32.MinValue instead of -1:

 

Comparison\<string\> bizarre = whatever;

and we compose it:

 

Comparison\<string\> reverseFirstByLength = Reverse(FirstByLength(bizarre));

Suppose two strings are equal in length and bizarre returns Int32.MinValue for those strings. Reverse should return a positive number, but -Int32.MinValue either throws an exception (in a checked context) or returns Int32.MinValue right back (in an unchecked context). Remember, there are more negative numbers that fit into an integer than positive numbers, by one.

The right implementation of Reverse is either to spell it out:

 

static Comparison\<string\> Reverse(Comparison\<string\> comparison)  
{  
  return (string x, string y) =\>  
  {  
    int result = comparison(x, y);  
    if (result \> 0) return -1;  
    if (result \< 0) return 1;  
    return 0;  
  };  
}

Or, as some commenters noted, to swap left and right:

 

static Comparison\<string\> Reverse(Comparison\<string\> comparison)  
{  
  return (string x, string y) =\> comparison(y, x);  
}

**Next time**: One more defect to spot.

