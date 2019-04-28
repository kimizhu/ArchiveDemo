# Null Is Not Empty

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/14/2009 10:47:00 AM

-----

Back when I started this blog in 2003, one of the first topics I posted on was the [difference between Null, Empty and Nothing](http://beta.blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx) in VBScript. An excerpt:

> Suppose you have a database of sales reports, and you ask the database "*what was the total of all sales in August*?" but one of the sales staff has not reported their sales for August yet. What's the correct answer? You could design the database to ignore the fact that data is missing and give the sum of the known sales, but that would be answering a different question. The question was not "*what was the total of all known sales in August, excluding any missing data*?" The question was "*what was the total of all sales in August*?" The answer to that question is "**I don't know -- there is data missing**", so the database returns Null.

This principle underlies the design of nullable value types in C\#. The reason that we have nullable value types at all is because there is a semantic difference between the null integer/decimal/double/whatever and the zeroes of those types. A zero means “I know that the quantity is zero”, a null means “I don’t know what the quantity is”.

This also explains why nulls propagate; if you add two nullable ints and one of them is null then the answer is null. Clearly ten plus “I don’t know” equals “I don’t know”, not ten.

The concept of “null as missing information” also applies to reference types, which are of course always nullable. I am occasionally asked why C\# does not simply treat null references passed to “foreach” as empty collections, or treat null strings as empty strings (\*). It’s for the same reason as why we don’t treat null integers as zeroes. There is a *semantic* difference between “the collection of results is known to be empty” and “the collection of results could not even be determined in the first place”, and we want to allow you to preserve that distinction, not blur the line between them. By treating null as empty, we would diminish the value of being able to strongly distinguish between a missing or invalid collection and and present, valid, empty collection.

Now, if for some odd reason you do wish to treat null collections the same as empty collections, that’s easy enough to do. You can simply use the null coalescing operator; that’s what it’s for:

 

foreach(Customer customer in customers ?? Enumerable.Empty\<Customer\>())

The ?? operator means “use the left hand side, unless if the left hand side is null, use the right hand side.” Handy, that.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) C\# does treat null strings as empty strings when concatenating them. See the comments for a discussion of this fact.

