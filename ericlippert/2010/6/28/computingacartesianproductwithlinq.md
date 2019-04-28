# Computing a Cartesian Product with LINQ

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/28/2010 8:06:00 AM

-----

And here we have [yet another post inspired by a question on StackOverflow](http://stackoverflow.com/questions/3093622): **how do you compute the Cartesian product of arbitrarily many sequences using LINQ? (\*)**

UPDATE: Ian Griffiths has an [interesting series of articles](http://www.interact-sw.co.uk/iangblog/2010/07/28/linq-cartesian-1) that approaches this question in considerably more depth than I do; check it out\!

First off, let's make sure that we know what we're talking about. I'll notate sequences as ordered sets {a, b, c, d,...}. The Cartesian product of two sequences S1 and S2 is the sequence of all possible two-element sequences where the first element is from S1 and the second element is from S2. So for example, if you have the sequences {a, b} and {x, y, z} then their Cartesian product is the sequence of two-element sequences {{a, x}, {a, y}, {a, z}, {b, x}, {b, y}, {b, z}}.

For simplicity's sake, let's assume that S1 and S2 are sequences of the same element type. We certainly could define a Cartesian product of a sequence of strings with a sequence of ints as a sequence of tuples of (string, int), but then it gets quite difficult to generalize this because the C\# generic type system does not handle arbitrarily-sized tuples particularly nicely.

LINQ has an operator specifically for making Cartesian products: in "fluent" syntax it is SelectMany, in "query comprehension" syntax it is a query with two "from" clauses:

var s1 = new\[\] {a, b};  
var s2 = new\[\] {x, y, z};  
var product =  
    from first in s1  
    from second in s2  
    select new\[\] { first, second };

  
We can of course generalize the Cartesian product to more than two sequences. The Cartesian product of n sequences {S1, S2, ... Sn} is the sequence of all possible n-element sequences where the first element is from S1, the second element is from S2, and so on.

There's a trivial case missing from that definition of course; what is the Cartesian product of zero sequences? Let's say that the Cartesian product of a sequence containing a single empty sequence, that is, { { } }. (See the comments for a justification of why this is a good idea; I had originally thought to use the empty sequence { }, but this is way better. Thanks to commenter Apollonius for the excellent suggestion.)

Note that this gives a reasonable definition for the Cartesian product of a single sequence. The Cartesian product of a sequence containing one sequence, say, {{a, b}}, is the sequence of all possible one-element sequences where the first (and only) element is from {a, b}. So the Cartesian product of {{a, b}} is {{a}, {b}}.

With LINQ you can make the Cartesian product of any number of sequences easily enough *if you know how many sequences there are to begin with*:

var product =  
    from first in s1  
    from second in s2  
    from third in s3  
    select new\[\] {first, second, third};

  
But what do you do if you do not know how many sequences there are at compile time? That is, how do you write the body of

public static IEnumerable\<IEnumerable\<T\>\> CartesianProduct\<T\>(this IEnumerable\<IEnumerable\<T\>\> sequences)

  
?

Well, let's reason using induction; that's almost always a good idea when working on recursively-defined data structures.

If sequences contains zero sequences, we're done; we just return { { } }.

How do we compute the Cartesian product of two sequences, say {a, b} and {x, y, z} again? We start by computing the Cartesian product of the first sequence. Let's make the inductive hypothesis that we have some way to do that, so we know its {{a}, {b}}. How do we combine {{a}, {b}} with {x, y, z} to produce the desired Cartesian product?

Well, suppose we go back to our original definition of the Cartesian product of two sequences to get some inspiration. The Cartesian product of {{a}, {b}} and {x, y, z} is the mess {{{a}, x}, {{a}, y}, {{a}, z}, {{b}, x}, {{b}, y}, {{b},z}} which is tantalizingly close to what we want. We do not want to only compute the Cartesian product of {{a}, {b}} and {x, y, z} by making a *sequence* containing {a} and x, we want to compute the Cartesian product by *appending* x to {a} to produce {a, x}\! Or, put another way, by *concatenating* {a} with {x}.

In code: suppose we already have an old Cartesian product, say {{a}, {b}}. We wish to combine it with sequence {x, y, z}:

var newProduct =  
    from old in oldProduct  
    from item in sequence  
    select old.Concat(new\[\]{item}};

  
And now we have a successful recursive case. If oldProduct is *any* Cartesian product then we can compute the combination of it with another sequence to produce a new Cartesian product.

Just to make sure: does this work with the base case? Yes. If we want to take the Cartesian product of { { } } with {a, b} then we concatenate { } with {a} and concatenate { } with {b} to get {{a}, {b}}.

Let's put it all together.

static IEnumerable\<IEnumerable\<T\>\> CartesianProduct\<T\>(this IEnumerable\<IEnumerable\<T\>\> sequences)  
{  
  // base case:  
  IEnumerable\<IEnumerable\<T\>\> result = new\[\] { Enumerable.Empty\<T\>() };  
  foreach(var sequence in sequences)  
  {  
    var s = sequence; // don't close over the loop variable  
    // recursive case: use SelectMany to build the new product out of the old one  
    result =  
      from seq in result  
      from item in s  
      select seq.Concat(new\[\] {item});   
    }  
  return result;  
}

  
That's fine, but we could actually be a bit fancier here if we wanted to. We are essentially using an *accumulator*. Consider a simpler case, say, adding up the total of a list of integers. One way to do that is to say "the accumulator starts at zero. The new accumulator is computed from the old accumulator by adding the current item to the old accumulator." If you have a starting value for an accumulator and some way to make a new accumulator from an old accumulator and the current item in the sequence then you can use the handy Aggregate extension method. It takes the starting value of the accumulator and a function that takes the last value and the current item and returns you the next value for the accumulator. The result is the final value of the accumulator.

In this case we'll start our accumulator off as the empty product, and every time through we'll "add" to it by combining the current sequence with the product so far. At every step of the way, the accumulator will be the Cartesian product of all the sequences seen so far.

static IEnumerable\<IEnumerable\<T\>\> CartesianProduct\<T\>(this IEnumerable\<IEnumerable\<T\>\> sequences)  
{  
  IEnumerable\<IEnumerable\<T\>\> emptyProduct = new\[\] { Enumerable.Empty\<T\>() };  
  return sequences.Aggregate(  
    emptyProduct,  
    (accumulator, sequence) =\>  
      from accseq in accumulator  
      from item in sequence  
      select accseq.Concat(new\[\] {item}));  
}

  
Now, a word to the wise here. Remember that **with LINQ the result of a query expression is a query that can deliver the results when asked, not the results of the query**. When we construct this accumulation, we're not actually computing the Cartesian product. We are computing a big complicated query that *when executed*, results in the Cartesian product. The query will be built eagerly, but executed lazily.

-----

(\*) The picky reader will note that strictly speaking there are two minor issues here. First, the Cartesian product on arbitrarily many products is properly called the "n-ary Cartesian product". Second, Cartesian products are defined on sets, not on sequences. In this post we make the obvious restriction of the n-ary Cartesian product from arbitrary, possibly infinite and unordered sets, to finite sequences.

