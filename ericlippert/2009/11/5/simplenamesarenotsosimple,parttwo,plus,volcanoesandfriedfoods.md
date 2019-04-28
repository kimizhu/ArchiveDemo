# Simple names are not so simple, Part Two, plus, volcanoes and fried foods

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/5/2009 6:52:00 AM

-----

I've returned from a brief vacation, visiting friends on the island of Maui. I'd never been to that part of the world before. Turns out, it's a small island in the middle of the Pacific Ocean, entirely made out of volcanoes. Weird\! But delightful.

The most impressive thing about the Hawaiian Islands for me was just how obvious were -- even to my completely untrained eyes -- the geomechanical and fluvial processes which shaped the landscape. The mountains and craters and river valleys and red sand beaches and easily-fractured rocks were very different from the (also somewhat volcanic) much older mountainous landscape I've lived in for the past decade.

Also quite amusing to me was learning to read and pronounce Hawaiian place names. It is all very logical once you know the system; before long I could easily pronounce signs like WAINAPANAPA STATE PARK -- wa-ee-napa-napa -- or PUUNENE AVENUE -- pu-oo-nay-nay -- or MAILIBEHANAMONOTANA STREET -- "Miley-Stewart-is-really-Hannah-Montana".

Many thanks to K and R and D for putting me and Leah up for a week; if you're going to Hawai'i and can stay with locals, I highly recommend it, particularly if they are awesome people. Everyone in Maui was awesome, with the exception of the rangers at (stunningly beautiful, even for Maui) Wa'inapanapa State Park, who are apparently consistently grumpy. As one Hawaiian, himself in the camping sector of the economy put it to me, "They do not have *the big aloha*".

The most *amusing* encounter was on the Hana Highway. There are numerous little stops along the way, where someone has erected a hut or parked a trailer and is selling coconuts, smoothies, banana bread, and so on. Hand-lettered signs, stunning natural beauty, middle of nowhere, you get the picture I'm sure. At one of the larger such stops there was a young fellow, probably in his late twenties, serving a variety of fried foods. It was mostly traditional American-style Chinese food, but also he had french fries, fish'n'chips, and so on. He was clearly not a native speaker of English, but spoke understandably with a strong accent. We were waiting behind a middled-aged woman with a typically midwestern American accent. Their conversation went something like this:

**Her:** I'm not very hungry, can I just get the fish without the chips?

**Him, not quite following her:** Half order?

**Her, louder**: How much without the fries?

This went back and forth for some time, both sides becoming increasingly frustrated by the communication breakdown, until:

**Her, even louder**: Can I speak to your manager?

Leah and K and I silently *boggled* -- there is no other word for it -- at each other for a moment. Where on earth did she imagine that a *manager* was going to emerge from? There was a counter, behind that, a trailer with a wok in it, behind that, *jungle*, and behind that, *huge jagged lava rocks* followed immediately by *the Pacific Ocean*. And what sort of *management structure* does she think one really needs to manage a single guy selling pineapple fried rice at the side of a highway? My conclusion: *people have strange beliefs.* Sometimes their beliefs cause them to leave in a huff with neither fish nor chips, even *when fish and chips are both plentiful and reasonably priced*. Hopefully she had better luck in Hana.

Anyway, enough travelogue. Regarding the puzzle from last time: the code is correct, and compiles without issue. I was quite surprised when I first learned that; it certainly looks like it violates our rule about not using the same simple name to mean two different things in one block.

The key is to understanding why this is legal is that the query comprehensions and foreach loops are specified as *syntactic sugars* for another program, and it is *that* program which is actually analyzed for correctness. Our original program:

 

static void Main()  
{  
  int\[\] data = { 1, 2, 3, 1, 2, 1 };  
  foreach (var m in from m in data orderby m select m)  
    System.Console.Write(m);  
}

is transformed into

 

static void Main()  
{  
  int\[\] data = { 1, 2, 3, 1, 2, 1 };  
  {  
    IEnumerator\<int\> e = ((IEnumerable\<int\>)(data.OrderBy(m=\>m)).GetEnumerator();  
    try  
    {  
      int m;  
      while(e.MoveNext())  
      {  
        m = (int)(int)e.Current;  
        Console.Write(m);  
      }  
    }  
    finally  
    {  
      if (e \!= null) ((IDisposable)e).Dispose();  
    }  
  }  
}  
   

There are five usages of m in this transformed program; it is:

1\) declared as the formal parameter of a lambda.  
2\) used in the body of the lambda; here it refers to the formal parameter.  
3\) declared as a local variable  
4\) written to in the loop; here it refers to the local variable  
5\) read from in the loop; here it refers to the local variable

Is there any usage of a local variable before its declaration? No.

Are there any two declarations that have the same name in the same declaration space? It would appear so. The body of Main defines a local variable declaration space, and clearly the body of Main contains, indirectly, two declarations for m, one as a formal lambda parameter and one as a local. But I said last time that local variable declaration spaces have special rules for determining overlaps. It is illegal for a local variable declaration space to directly contain a declaration such that another nested local variable declaration space contains a declaration of the same name. But an outer declaration space which indirectly contains two such declarations is not  an error. So in this case, no, there are no local variable declarations spaces which directly contain a declaration for m, such that a nested local variable declaration space also directly contains a declaration for m. Our two local variable declarations spaces which directly contain a declaration for m do not overlap anywhere.

Is there any declaration space which contains two inconsistent usages of the simple name m? Yes, again, the outer block of Main contains two inconsistent usages of m. But again, this is not relevant. The question is whether any declaration space directly containing m has an inconsistent usage. Again, we have two declaration spaces but they do not overlap each other, so there's no problem here either.

The thing which makes this legal, interestingly enough, is the generation of the loop variable declaration logically within the try block. Were it to be generated outside the try block then this would be a violation of the rule about inconsistent usage of a simple name throughout a declaration space.

