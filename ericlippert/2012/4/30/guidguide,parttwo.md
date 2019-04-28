# GUID guide, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/30/2012 7:11:00 AM

-----

So how is it that a GUID can be guaranteed to be unique without some sort of central authority that ensures uniqueness, a la the ISBN system?

Well, first off, notice that the number of possible GUIDs is *vastly* larger than the number of possible ISBNs. Because the last of the thirteen digits is a checksum, there are only 10<sup>12</sup> possible ISBNs. That is about a hundred unique ISBNs for every person on earth. That's almost exactly 2<sup>40</sup>, so you could represent an ISBN by a 40 bit number (again, ignoring the checksum). There are 2<sup>128</sup> possible GUIDs; that's about 40 billion billion billion unique GUIDs for every person on earth. This alone gives us the intuition that it ought to be pretty easy to ensure that two of them never collide; there are a *lot* of GUIDs to choose from\!

There are a number of possible strategies for making a unique GUID, and in fact information about the strategy used is encoded in the first four bits of the third "group"; almost every GUID you see will be of the form {xxxxxxxx-xxxx-1xxx-xxxx-xxxxxxxxxxxx} or {xxxxxxxx-xxxx-4xxx-xxxx-xxxxxxxxxxxx}.

If there is a one in that place then the algorithm used to guarantee uniqueness is essentially a variation on the ISBN strategy. The GUID is guaranteed to be unique "in space" by choosing some of the bits as the MAC address of the network card in the machine. (The tricky problem of ensuring that no two network cards in the world have the same MAC address is [solved somehow by someone else](http://en.wikipedia.org/wiki/Organizationally_Unique_Identifier); how that problem is solved, we don't particularly care. The cost of solving that problem is passed on to you, the consumer, in the purchase cost of the network card.)

(**UPDATE**: [Larry Osterman](http://blogs.msdn.com/b/larryosterman/) pointed out to me that of course, this MAC address solution is not foolproof. First, you could deliberately or accidentally set your MAC address to an already-used value. Second, a hardware manufacturer might forget to set the address at all and have it be all zeros. Third, two virtual machines might be using the same physical network card and those virtual machines could be generating GUIDs at exactly the same time fast enough to cause collisions.)

We know that we can rely on this as a source of uniqueness in space. Most of the remaining bits are a high-resolution timestamp. Therefore every generated GUID is unique in both space and time, and is therefore globally unique.

This system does in practice have a few weak spots. The most obvious one is that it does not work well if the machine does not have a network card\! Version-one GUIDs generated on machines without network cards are not guaranteed to be unique. The less obvious one is that there is a slim chance that two GUIDs could be generated "at the same time". Perhaps two GUID generators were running on two different processors in the same machine at the same time. Or perhaps a GUID was generated, the clock in the machine was deliberately "turned back", and the same GUID was then generated again, just by bad luck. There are a few "extra" bits in a GUID that are used to mitigate these timing problems, so in practice they don't arise.

There are a number of interesting consequences of this algorithm. The first is that such GUIDs are almost completely *non-random*. Many people seem to be of the mistaken belief that GUIDs are a source of *randomness*, when in fact they are only guaranteed to be a source of *uniqueness*.

The second is that it is possible for GUIDs generated on a particular machine with this algorithm to be *monotone increasing*. This is actually a very nice property for GUIDs to have; GUIDs are often used as primary keys in databases, and it can be much cheaper to insert a large number of rows into an indexed-by-primary-key database table if the rows are already sorted and you're always inserting *after* every previous entry. This again demonstrates that using a sequence of GUIDs as a sequence of random 128 bit numbers is a terrible idea; random numbers are typically not monotone increasing\!

The third interesting consequence is that code or documents that contains a GUID generated with this first algorithm **contain information that uniquely identifies the machine that was used to create the GUID**. Just as a skilled reader can determine interesting facts about a book from its ISBN, so too can a skilled reader determine when and where a GUID was generated if it has a one as its thirteenth hex digit. This fact was used when tracking down and prosecuting the author of the famous [Melissa](http://en.wikipedia.org/wiki/Melissa_\(computer_virus\)) virus. (We'll discuss the implications of this in more detail in an upcoming episode.)

The fourth interesting consequence is that **no proper subsequence of the bits of a GUID have the global uniqueness property**, [as Raymond pointed out back in 2008](http://blogs.msdn.com/b/oldnewthing/archive/2008/06/27/8659071.aspx). And indeed, we have no reason to expect that a smaller set of bits would have the same properties as a larger set of bits\! You don't expect to be able to saw one aircraft in half and end up with two things that both fly.

**Next time** we'll talk about GUIDs that have a four in their thirteenth hex digit; they use a completely different technique for ensuring uniqueness.

