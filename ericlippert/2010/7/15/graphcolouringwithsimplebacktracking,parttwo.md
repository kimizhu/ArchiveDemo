# Graph Colouring With Simple Backtracking, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/15/2010 8:09:00 AM

-----

Before I begin a quick note: congratulations and best wishes to David Johnson, currently the president of my alma mater, the University of Waterloo. The Queen has appointed him to be the next Governor General of Canada come this October. For those of you unfamiliar with the Canadian political structure, Queen Elizabeth is the sovereign ruler Canada; the Governor General acts as her direct representative in Canada and therefore has the (mostly ceremonial, but some real) powers of a head of state.

Moving on\!

OK, we've got our graph class. Our basic algorithm for assigning colours will be to associate with each node in the graph a set of "possible colours", and then gradually reduce that set for each node until either we've proved that there is no possible colouring, or we've found one that assigns exactly one colour to each node. Therefore we will need a type that represents a set of possible colours.

First off, what type should represent a colour? There are already Color enums in the BCL, but is that really what we want? We're probably going to want to ask questions like "is there a colouration of this graph that uses only three colours?", rather than "is there a colouration of this graph that uses only green, blue and orange?"  The actual colours are kind of irrelevant. Let's say that a colour is simply an integer from zero to max-1. We can always make a mapping from integers to real colours if we want to.

Note that this design decision immediately introduces a point of possible confusion; last time we said that nodes in a graph were also represented as integers. If I were particularly concerned about that, I might devise two structs that each wrap an integer, say, GraphNode and GraphColour. Then I could let the compiler tell me if I were to accidentally assign something that is logically a colour to a variable that is logically a node.

This is a tough call. Basically we are trading off the time and effort of implementing and maintaining the unnecessary wrapper structs against the possibility of time and effort lost to accidental bugs. Does the cost justify the benefit? My feeling is in this case, no. I'm going to not implement these wrapper structs this time.

OK, so a colour is a plain integer. I want to represent a set of plain integers. A HashSet\<int\> seems like a reasonable off-the-shelf choice, but it has a few problems. First off, it is mutable; part of the point of this exercise is to program in an immutable style. Second, it's heavyweight; in order to maintain a set of integers it is allocating arrays and hash buckets and all kinds of stuff.

If we're willing to constrain the problem we can do a lot better than a HashSet\<int\> to represent a set of integers. Suppose for example that we say that we're never going to try to colour a graph with more than 32 colours, which seems reasonable. Given that constraint, we can represent a set of integers in a single 32 bit integer, by simply using the integer as a bit set. Integers are already immutable, so it seems natural to make an immutable set out of an integer.

Now, of course if that's our approach then we don't even need a type that wraps an integer. We could just use an integer. But, unlike in our example of the graph nodes and graph colours, we actually have operations we want to perform on this type, like "enumerate all your members". This is much more like a real abstract data type, so it's justifiable to make a real type out of it rather than simply using an integer directly.

// A super-cheap immutable set of integers from 0 to 31 ;  
// just a convenient wrapper around bit operations on an int.  
internal struct BitSet : IEnumerable\<int\>  
{  
  public static BitSet Empty { get { return default(BitSet); } }  
  private readonly int bits;  
  private BitSet(int bits) { this.bits = bits; }  
  public bool Contains(int item)  
  {  
    Debug.Assert(0 \<= item && item \<= 31);   
    return (bits & (1 \<\< item)) \!= 0;  
  }  
  public BitSet Add(int item)  
  {   
    Debug.Assert(0 \<= item && item \<= 31);   
    return new BitSet(this.bits | (1 \<\< item));  
  }  
  public BitSet Remove(int item)  
  {   
    Debug.Assert(0 \<= item && item \<= 31);   
    return new BitSet(this.bits & ~(1 \<\< item));  
  }  
  IEnumerator IEnumerable.GetEnumerator() { return this.GetEnumerator(); }  
  public IEnumerator\<int\> GetEnumerator()  
  {  
    for(int item = 0; item \< 32; ++item)  
      if (this.Contains(item))  
        yield return item;  
  }  
}

Let's again go through this line by line.

As with the graph class, I've made the type internal. Since I know that I'm the only one using this, I can be a bit more cavalier about not doing bounds checking with nice exceptions and whatnot. If this were a public type in the BCL then I'd have to do much more "retail" bounds checking to ensure that the items actually are integers from 0 to 31; instead, I'll just do that checking with debug mode assertions.

An immutable set of integers can logically be treated as a value, and this type is very small in terms of the data it contains, so I've made this a value type.

Unlike the graph class, here I am emphasizing the implementation detail of the set by naming the class BitSet instead of, say, SmallIntegerSet. Am I inconsistent? Yes, I contain multitudes. It feels like the right thing to do here is to emphasize that this is fundamentally a mechanism, rather than the carrier of semantics. I'm not sure why that is.

I could implement a more full-on interface like ICollection\<int\>, but that includes operations that I don't want, like the ability to mutate the collection. Rather than use an off-the-shelf interface in a confusing, off-label manner, I'll just implement the standard interface that I can implement correctly, IEnumerable\<int\>, and then implement my own Add and Remove methods that do the right thing.

I'm using the standard immutable object pattern of providing a static way to get at the empty collection. This is unnecessary; default(BitSet) gives you the right thing. But it reads much more nicely to say BitSet.Empty; it is clear from the code that you're getting an empty collection.

I've made the private data a signed integer rather than an unsigned integer. It's marginally safer to represent a bit vector as a uint because you run less risk of having a problem with sign bit issues. However, you also then have to do casts all over the show, which is ugly. I say that the math here is simple enough that we can verify that it is correct without worrying too much about sign issues.

The constructor which sets the internal state is private; this deprives the caller of the power of setting many bits at once. But if the caller has an int that has many bits set at once, then why are they using the BitSet in the first place? Apparently they're already willing to do bit twiddling in their own code to set those bits. Again the YAGNI principle says to not provide unneeded functionality.

The Contains, Add and Remove methods, as mentioned before, do not verify the validity of their arguments because the type is only used internally. Debug assertions tell the caller when an invariant expected by the type has been violated.

You might wonder why the GetEnumerator method does a loop with a yield return, rather than being implemented as

  public IEnumerator\<int\> GetEnumerator()  
  {  
    return   
      from item in Enumerable.Range(0, 32)   
      where this.Contains(item)   
      select item;  
  }

Do you see why I chose to use a loop for this one, despite my stated desire to eschew loops in favour of queries?

Notice that I choose to not implement special versions of sequence operators that are extension methods, even though I could. For example, there's no "Count" property on this thing. The naive implementation of Count() will call GetEnumerator, which as we've seen enumerates all 32 possibilities. But the count of the bitset is the [Hamming Weight](http://en.wikipedia.org/wiki/Hamming_weight), and there are many well-known algorithms for efficiently calculating the Hamming Weight of a bit vector. We could for example use the algorithm commonly attributed to Kernighan (though it has been discovered independently many times throughout the history of computer programming.) This algorithm clears the low bit until they're all cleared, and tells you how many times it had to do that:

  public int Count()  
  {  
    int remaining = this.bits;  
    int count = 0;  
    while(remaining \!= 0)  
    {  
        remaining &= (remaining - 1);  
        count++;  
    }  
    return count;  
  }

The cost of this is proportional to the number of set bits, rather than always going through all 32. But who cares? 32 is a small number. Again, let's not add this complexity unless profile-driven empirical data suggests that naively counting the bitset entails an unacceptably high cost.

Notice that in contrast I *did* choose to implement Contains() myself rather than using the extension method. Had I used the extension method then obviously I would not have been able to use Contains() inside GetEnumerator; we would have gotten a stack overflow. I would have had to put the bit twiddling logic in GetEnumerator, and since I was going to have to write the bit twiddling logic *anyway*, it seemed reasonable to put it in its own method for general use.

Once more we've built a trivial little piece of code, but to do so, had to make many design decisions along the way. The fundamental question that has driven the entire design is "is it reasonable that we'll never have more than 32 colours?" We could trivially expand that to 64 by using a long instead of an int, but going farther than that would be a lot trickier.

Next time, let's actually solve a slightly harder problem; let's make an algorithm that actually solves colouring problems.

