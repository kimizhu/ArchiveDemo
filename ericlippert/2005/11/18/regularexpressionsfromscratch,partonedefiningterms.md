# Regular Expressions From Scratch, Part One: Defining Terms

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/18/2005 1:00:00 PM

-----

Over the years that I've been writing this blog some of the most positive feedback I've received has been for those entries where I've explored fundamental concepts in computer science. I thought that I might take that to its logical extreme, and do a series on what exactly a "regular expression" is.

Most script developers are familiar with regular expressions - they're those dense, hard-to-read, patterns that you can use to write self-obfuscating code that does string matching. I'd like to start by throwing out everything that we know about practical, real-world regular expressions and start from set theory, of all things.

This is going to be a very long series I think. But it will build character\!

Here's what I'm going to assume:

  - We have an *intuitive* understanding of **sets**. Sets are collections of stuff. They can be empty, finite, infinite, you name it. They can contain other sets. I am not going to start with the axioms of set theory, but trust me, we could justify our intuitive understanding of sets axiomatically if we wanted to.

  - We have an intuitive understanding of the set of **natural numbers** ℕ = {0, 1, 2, 3, …}

  - We can create arbitrary sets of **symbols**. For clarity, throughout this series I'll be displaying symbols in a blue fixed-width typeface. We could have a set of symbols S = {a, b, c, +, -, 0, 1, &} for instance.

  - We have an intuitive understanding of **finite sequences**. Remember, sets are by definition unordered, may be infinite, and contain no duplicates. A finite sequence is like a set but is finite, has an order, and may contain duplicates. A finite sequence may be empty. (Again, we could derive a suitable definition of a finite sequence from axiomatic set theory, but I won't.) To distinguish finite sequences from sets, I'll use curly braces for sets and round parentheses for finite sequences, but not for long.

That's not starting with much. Let's see what we can do.

**Definition 1**: An **alphabet** is a finite set of symbols.

For example, we could have the alphabet of capital Roman letters R = {A, B, C, …, Z} or the alphabet of binary digits B = {0, 1}.

The empty alphabet is a pretty boring alphabet, but it is a legal alphabet.

**Definition 2**: a **string** is a finite sequence of symbols drawn from a given alphabet.

It is a pain in the rear to continually say "(b, b, c) is a string over alphabet {a, b, c}" so for the vast majority of cases, I'll be writing strings all run together like this: bbc and assuming that you can mentally treat this as a sequence.

I'll also be assuming that you can figure out the alphabet for a given string from context. If the alphabet is ever unclear then I'll call it out specifically.

I will sometimes use "variables" to refer to strings and languages. You'll be able to distinguish variables from symbols because symbols are always written in blue fixed-width font. I shall endeavour to also avoid saying something confusing such as "Suppose b = bbc" though as we'll see later, this will become unavoidable when we talk about regular expressions.

There's no good way to show the empty string, not without getting into a whole lot of issues with quotation marks. I'm hereby declaring that unless I say otherwise, the variable e represents the empty string in whatever alphabet we're presently talking about.

Note that the empty string is the only string that can be formed from the empty alphabet.

I'll almost always use the variable S to represent an alphabet and u, v and w to represent strings.

**Definition 3**: Consider an alphabet S. The **Kleene Closure** of S is the set of all strings that can be formed from that alphabet and will be written as **S\***.

For example, let S = {0, 1}. Then S\* = {e, 0, 1, 00, 01, 10, 11, 000, …}. That is, every string of finite length that can be formed from only 0 and 1, including the empty string.

S\* is of course an infinite set, though I hasten to emphasize that no member of S\* is infinitely long, because strings are by definition finite sequences.

Here comes our most important definition today:

**Definition 4**: A **language** over an alphabet S is any subset of S\*.

This is a rather broad definition of "language" -- any subset *whatsoever* of the set of all finite strings in any alphabet forms a language.

Next time we'll look at some examples of languages, both sensible and crazy.

