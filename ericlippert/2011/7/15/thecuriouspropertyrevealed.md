# The curious property revealed

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/15/2011 7:26:00 AM

-----

Today is the fifteenth anniversary of my first day of full time work here at Microsoft. Hard to believe it has been a decade and a half of writing developer tools. I am tremendously fortunate to be able to work with such a great team on such a great toolset for such great customers. I'm looking forward to the next decade and a half\!

Now back to our regularly scheduled puzzling.

There are two curious properties of[the string I mentioned last time](http://blogs.msdn.com/b/ericlippert/archive/2011/07/12/what-curious-property-does-this-string-have.aspx). The first somewhat curious property is that this **string has the same hash code as the empty string.** A reader posted a comment to that effect within a half an hour of the blog post going up.

Of course, there are only four billion possible hash codes and way more than four billion possible strings, so [there have got to be collisions in there somewhere](http://blogs.msdn.com/b/ericlippert/archive/2010/03/22/socks-birthdays-and-hash-collisions.aspx). There are infinitely many strings that have this property; this just happens to be a particularly short one.

What is much more interesting is that **any number of concatenations of this string yields a string that *also* has the same hash code as the empty string**\! A number of readers discovered this property.

I hinted that the curious property was one shared by a common string. And indeed, the property of *having the same hash code for arbitrarily many concatenations* is also shared with the empty string. Ha ha\!

I noted in my first hint that this property was platform-dependent; you need to be using a non-debug 32 bit version of CLR v4. [As we document in the MSDN page, the string hashing algorithm is subject to change at any time](http://msdn.microsoft.com/en-us/library/system.string.gethashcode.aspx), and in fact [has changed in the past](http://blogs.msdn.com/b/ericlippert/archive/2005/10/24/482447.aspx). The debug version of the CLR changes its string hashing algorithm **every day**, so as to be sure that none of our internal infrastructure depends on the hash code algorithm remaining stable.

What gives this particular string this bizarre property?

The internals of the hashing algorithm are pretty straightforward; basically, we start with two integers in a particular state, and then mutate those integers in some way based on successive bytes of the string data. The algorithm works on considering two characters (four bytes) of string data at a time for efficiency. The transformation consists of some bit shifts and adds and whatnot, as you'd expect.

You can think of the hashing algorithm as a finite state machine with 64 bits of state. Each four bytes of input from the string changes the 64 bits of state. The final hashed result is derived from those 64 bits of state at the point where the input stream ends.

This particular sequence of four bytes - A0 A2 A0 A2 - has the property that it *transforms the start state bits into themselves*. As long as that input keeps coming in, the internals of the hash calculation *never* get out of the start state, and therefore must always output the same hash code as that of the empty string.

If you have a function f, and a value x such that f(x) = x, the value x is called a "fixed point" of f. What I've found here is technically not exactly a fixed point of the state transition function, but it has the same sort of flavour as a fixed point. Fixed points in various forms come up a lot in computer science and mathematics; they have many interesting and useful properties which I might go into in later fabulous adventures. Today though I want to think a bit about the security implications.

Essentially what we have here is a way of constructing arbitrarily many strings that all have the same hash code. (\*) If a hostile client can feed many such strings into a hash table on a benign server, the attacker can essentially mount a denial-of-service attack against the server that uses the hash table. The hash table will be unable to hash the strings into different buckets, and therefore it will get slower, and slower, and slower.

It's not much of an attack, but it is an attack. (\*\*)

As always, when thinking about how to mitigate attacks, think about "defense in depth". If you think you might be vulnerable to this sort of attack, make the attacker do several impossible things, not just one, to pull off the attack. For example:

\* Monitor inputs to ensure that one user isn't sending an unusually huge amount of data for processing; throttle them back if they are.

\* Filter inputs for unusual characters; attacks on hash tables are likely to involve out-of-the-ordinary data that has been specially crafted. Most people aren't going to be sending strings to a database full of Yi Script characters.

\* Monitor system performance, looking for slowdowns that might be indicative of DoS attacks

\* Build a custom hash table that uses its own string hashing algorithm. When you make a new hash table, randomize the starting state and the transformation of the hash.

And so on. Of course, this is just a sketch of a few ideas; if you are genuinely worried about such an attack, it is quite tricky to actually design and implement an attack-resistant hash table. Consult an expert. (I am not an expert on this\!)

\-----------

(\*) Of course there are plenty of ways to come up with hash collisions; like I said before, there are infinitely many hash collisions since there are only four billion possible hash codes. This is just a particularly cheap and easy way to come up with hash collisions.

(\*\*) And of course an attack by hostile clients on a slow hash table is just a special case of "attacking" yourself by accidentally hitting a slow code path due to a buggy design, [as I learned the hard way.](http://blogs.msdn.com/b/ericlippert/archive/2003/09/19/53054.aspx)

