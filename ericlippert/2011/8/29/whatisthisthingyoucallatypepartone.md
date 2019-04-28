# What is this thing you call a "type"? Part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/29/2011 7:33:00 AM

-----

(Eric is out camping; this posting is prerecorded. I'll be back in the office after Labour Day.)

The word "type" appears almost five thousand times in the C\# 4 specification, and there is an entire chapter, chapter 4, dedicated to nothing but describing types. We start the specification by noting that C\# is "type safe" and has "a unified type system" (\*). We say that programs "declare" types, and that declared types can be organized by namespace. Clearly types are incredibly important to the design of C\#, and incredibly important to C\# programmers, so it is a more than a little bit surprising that nowhere in the eight hundred pages of the specification do we ever actually define the word "type".

We sort of assume that the developer reading the specification already has a working understanding of what a "type" is; the spec does not aim to be either a beginner programming tutorial or a mathematically precise formal language description. But if you ask ten working line-of-business developers for a formal definition of "type", you might get ten different answers. So let's consider today the question: what exactly is this thing you call a "type"?

A common answer to that question is that a type consists of:

\* A name  
\* A set (possibly infinite) of values

And possibly also:

\* A finite list of rules for associating values not in the type with values in the type (that is, coercions; 123.45 is not a member of the integer type, but it can be coerced to 123.)

Though that is not a terrible definition as a first attempt, it runs into some pretty nasty problems when you look at it more deeply.

The first problem is that of course not every type needs to have a name; C\# 3 has anonymous types which have no names by definition. Is "string\[\]\[\]" the name of a type? What about "List\<string\[\]\[\]\>" -- does that name a type? Is the name of the string type "string" or "String", or "System.String", or "global::System.String", or all four? **Does a type's name *change* depending on where in the source code you are looking?**

This gets to be a bit of a mess. **I prefer to think of types as logically not having names at all**. Program fragments "12" and "10 + 2" and "3 \* 4" and "0x0C" are not *names* for the number 12, they are *expressions* which happen to all *evaluate* to the number 12. That number is just a number; how you choose to notate it is a fact about your notational system, not a fact about the number itself. Similarly for types; the program fragment "List\<string\[\]\[\]\>" might, in its context, evaluate to refer to a particular type, but that type need not have that fragment as its *name*. It has no name.

The concept of a "set of values" is also problematic, particularly if those sets are potentially infinite "mathematical" sets.

To start with, suppose you have a value: how do you determine what its type is? There are infinitely many sets that can contain that value, and therefore infinitely many types that the value can be\! And indeed, if the string "hello" is a member of the string type, clearly it is also a member of the object type. How are we to determine what "*the*" type of a value is?

Things get even more brain-destroying when you think, oh, I know, I'll just say that every type's set of values is defined by a "predicate" that tells me whether a given value is in the type or not. That seems very plausible, but then you have to think about questions like "are types themselves values?" If so, then "the type whose values are all the types that are not members of themself" is a valid predicate, and hey, we've got Russell's Paradox all over again. (\*\*)

Moreover, the idea of a type being defined by a predicate that identifies whether a given value is of that type or not is quite dissimilar to how we typically conceive of types in a C\# program. If it were, then we could have "predicate types" like "x is an even integer" or "x is not a prime number" or "x is a string that is a legal VBScript program" or whatever. We don't have types like that in the C\# type system.

So if a type is not a name, and a type is not a set of values, and a type is not a predicate that determines membership, what is a type? We seem to be getting no closer to a definition.

**Next time**: I'll try to come up with a more workable definition of what "type" means to both compiler writers and developers.

\------------

(\*) An unfortunate half-truth, as pointer types are not unified into the type system in the version of the C\# language which includes the "unsafe" subset of functionality.

(\*\*) When Russell discovered that his paradox undermined the entire arithmetic theory of Frege, he and Whitehead set about inventing what is now known as the "ramified theory of types", an almost impossible-to-understand theory that is immune to these sorts of paradoxes. Though the mathematical underpinnings of type theory/set theory/category theory/etc are fascinating, I do not understand them nearly well enough to go into detail here. Remember, what we're looking for is a definition of "type" that is amenable to doing practical work in a modern programming language.

