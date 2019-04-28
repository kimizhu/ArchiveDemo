# Bad Metaphors

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/13/2012 9:15:00 AM

-----

The standard way to teach beginner OO programmers about classes is to make a metaphor to the real world. And indeed, I do this all the time in this blog, usually to the animal kingdom. A "class" in real life codifies a commonality amongst a certain set of objects: mammals, for example, have many things in common; they have backbones, can grow hair, can make their own heat, and so on. A class in a programming language does the same thing: codifies a commonality amongst a certain set of objects via the mechanism of inheritance. Inheritance ensures commonalities because, as we've already discussed, "inheritance" by definition means "all (\*) the members of the base type are also members of the derived type".

Inheritance relationships amongst classes (\*\*) are usually designed to model "**is a special kind of**" relationships. A giraffe is a special kind of mammal, so the class Giraffe inherits from the class Mammal, which in turn inherits from Animal, which inherits from Object. And that's great; this clearly represents "is a special kind of" relationships. I have always, however, had a problem with the fundamental metaphor of "inheritance". Why "inheritance"? You inherit genetic information, property, and if you're a [titular lord](http://en.wikipedia.org/wiki/Christopher_Guest), your peerage, from your parents. And if you make a diagram of a class hierarchy, it looks a bit like a "family tree" in which the derived class is the "child" of the base "parent" class. And indeed, people often speak of the base class as the "parent" class of a "child" derived class, particularly when speaking to beginners.

But **the "parent-to-child inheritance" metaphor is awful**. A giraffe is not "a child of mammal"; a giraffe is a child of Mr. and Mrs. Giraffe. A "child" is not "a special kind of parent". In reality, you only inherit half your genetic makeup from each parent, and you can inherit real property from any relation, or for that matter, from any non-relation. In programming languages you only "inherit" from related types, and you inherit all their members (\*). In reality, everyone has two parents (\*\*\*), but in programming languages some languages allow inheritance from arbitrarily many "parents", some allow exactly one. In reality, a single, specific person inherits specific property from a single, specific parent, and two different children can have entirely different inheritances from their parent; in programming languages, the "inheritance" relationship does not apply to individual objects, and every child inherits exactly the same thing from the parents. And in reality you only inherit real property when the decedent is *dead*\!

But wait, it gets worse. The parent-child metaphor is *ambiguous* in any language that supports both *lexical nesting* and *nominal subtyping* of classes:

class B\<T\>  
{  
    class D\<U\> : B\<U\> { }  
}

Quick, what type is the "parent" of type B\<string\>.D\<int\>? Is it B\<T\> or B\<string\> or B\<int\>? That type is *lexically* inside B\<T\>, *logically* inside of B\<string\>, and *derived from* B\<int\>; which of those three is its *parent*? If you drew a graph showing either lexical or logical containment relationships, it would form a graph that looks every bit as much like a "family tree" as the graph showing inheritance relationships. And lexical containment allows access to all the properties of the container from the contained type, even including not-inherited and normally inaccesible members like private constructors\! It is not at all clear that one kind of "parentage" is actually more "parent-like" than any other.

As we've seen before, having multiple different "parent" relationships for a given type can make for [some extremely confusing code](http://blogs.msdn.com/b/ericlippert/archive/2007/07/27/an-inheritance-puzzle-part-one.aspx). We have to be extraordinarily careful when writing the specification and the compiler to ensure that we unambiguously describe precisely the relationship we wish to describe. I therefore try hard to avoid "parent-child" metaphors entirely; it is much more clear when writing an example to describe the type relationship as "base type and derived type", rather than "parent type and child type".

-----

(\*) Excepting constructors and destructors.

(\*\*) I'm going to stick to talking about class-based inheritance here; my criticisms apply equally well to interface-based inheritance but I don't want to open the can of worms that is all the subtle differences between class and interface inheritance. And I've never much liked the inheritance metaphor on interfaces anyways; a "contractual obligation" metaphor is better.

(\*\*\*) Assuming that we're talking about members of a sexually reproducing species.

