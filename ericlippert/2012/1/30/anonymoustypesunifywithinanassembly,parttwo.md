# Anonymous Types Unify Within An Assembly, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/30/2012 8:20:00 AM

-----

Last time I noted that any two usages of "the same" anonymous type within an assembly actually unify to be the same type. By "the same" we mean that the two anonymous types have the same property names and types, and that they appear in the same order. new {X = 1, Y = 2 } and new { Y = 2, X = 1 } do not unify to a single type.

Why is that? You'd think that we could make these unify.

The trouble is that doing so causes more problems than it mitigates. An anonymous type gives you a convenient place to store a small immutable set of name/value pairs, but it gives you more than that. It also gives you an implementation of Equals, GetHashCode and, most germane to this discussion, ToString. (\*)

Imagine, for instance, that you have written a bunch of LINQ queries in your code that extract data from a table using an anonymous type. As part of your unit testing, you dump the results of the query as a string out to a file and compare it against a known baseline. Maybe you have hundreds of such tests. And then one day, someone in a completely different part of the code happens to write a LINQ query that has "the same" anonymous type, but with the properties in a different order. We have to pick some order for the properties to be written out by "ToString", and there is no telling which one we'd pick if forced to choose. It seems very strange that using an anonymous type in one part of the program would cause tests to fail in a completely different part of the program.

Well then, you might say, you could solve this problem by canonicalizing the implementation of ToString. Always write out the properties in alphabetical order, say. But that is hardly an attractive solution. First off, whose alphabetical order? There are dozens of different alphabetical orders, depending on what location in the world you are in. Should we choose the alphabetical order of the developer? The current user? The "culture neutral" order? Assuming we could solve that problem satisfactorily, we'd still be disappointing most users. Developers have a reasonable expectation that ToString will give them the properties in the order they appear in the source code.

Another option would be to not implement ToString for you at all. That is to say, remove a useful and relatively commonly used feature (dumping data to a string for testing or debugging) in order to effectively implement a less useful, rarely used feature (unification of types).

-----

(\*) We give you Equals and GetHashCode so that you can use instances of anonymous types in LINQ queries as keys upon which to perform joins. LINQ to Objects implements joins using a hash table for performance reasons, and therefore we need correct implementations of Equals and GetHashCode.

