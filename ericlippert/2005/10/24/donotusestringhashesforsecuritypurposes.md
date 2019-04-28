# Do not use string hashes for security purposes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/24/2005 10:00:00 AM

-----

A recent question I got about the .NET CLR's hashing algorithm for strings is apropos of [our discussion from January on using salted hashes for security purposes](http://blogs.msdn.com/b/ericlippert/archive/tags/salt/). The question was basically *"my database of password hashes doesn't seem to work with .NET v2.0, what's up with that?"*

To make a long story short, the answer is **under no circumstances should you use String.GetHashCode for security purposes**. It was not designed for anything other than hash table balancing. If it hurts when you do that then stop doing it\!

The slightly longer version goes like this. Suppose you want to store some secrets in a database, but you only need to be able to confirm that the user knows the secret. As I discussed in my series on salted hashes, a hash is a commonly used tool for this task because a cryptographic hash has some nice properties. Namely, it is a fixed number of bits (in the 100's of bits range), small changes to input produce huge changes in output, and it is very difficult to go from the hash back to the original secret. Another nice property that I didn't call out in my earlier article is that there are industry-standard hash algorithms where you can be reasonably guaranteed that any two implementations will produce the same results when given the same set of bits.

The .NET CLR string hash algorithm has none of these nice properties, and therefore is completely unsuitable for a cryptographic hash function. Specifically:

1\) The string hash algorithm was designed to be blindingly fast rather than hard to run backwards, so it is likely that a mathematically astute attacker will be able to rapidly deduce facts about the input knowing only the hash.

2\) Worse, being only 32 bits, using brute force to find a message that produces a given hash becomes doable in an afternoon with a PC rather than a trillion years.

3\) The string hash algorithm is not an industry standard.

4\) [GetHashCode is not guaranteed to produce the same behaviour between versions](http://msdn.microsoft.com/en-us/library/system.string.gethashcode.aspx). And in fact it does not. The .NET 2.0 CLR uses a different algorithm for string hashing than the .NET 1.1 CLR, and 64 bit versions of the CLR use different algorithms than the 32 bit versions.

If you are saving .NET 1.1 CLR hash values in a database then you will not be able to match them when you upgrade to 2.0. The hash algorithm was specifically *not* designed to be forward/backward compatible and we called that out in the documentation. Do not rely on compatibility when we specifically call out in the documentation that you should not rely on version-to-version identical output for a function.

Please don't do that; you're just asking for a world of hurt if you do. Use SHA2 or some other algorithm designed for that purpose. Yes, I know that weaknesses have been discovered in some standard hashing algorithms, but they are still orders of magnitude better than a hash designed for hash table balancing\!

