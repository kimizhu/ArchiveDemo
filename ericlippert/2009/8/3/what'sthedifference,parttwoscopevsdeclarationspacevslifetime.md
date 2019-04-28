# What's The Difference, Part Two: Scope vs Declaration Space vs Lifetime

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/3/2009 10:08:00 AM

-----

"Scope" has got to be one of the most confusing words in all of programming language design. People seem to use it casually to mean whatever is convenient at the time; I most often see it confused with *lifetime* and *declaration space*. As in "the memory will be released when the variable goes out of scope".

In an informal setting, of course it is perfectly acceptable to use "scope" to mean whatever you want, so long as the meaning is clearly communicated to the audience. In a more formal setting, like a book or a language specification, it's probably better to be precise.

The difference between scope and declaration space in C\# is subtle.

The **scope** of a named entity is the **region of program text** in which it is legal to refer to that entity **by its unqualified name**.

There are some subtleties here. The implication does not "go the other way" -- it is not the case that if you can legally use the unqualified name of an entity, that the name refers to that entity. Scopes are allowed to overlap. For example, if you have:

 

class C  
{  
  int x;  
  void M()  
  {  
    int x;  
  }  
}

then the field is in scope throughout the entire body text of C, including the entirity of M. Local variable x is in scope throughout the body of M, so the scopes overlap. When you say "x", whether the field or the local is chosen depends on where you say it.

A **declaration space**, by contrast, is a region of program text in which **no two entities are allowed to have the same name**. For example, in the region of text which is body of C *excluding the body of M*, you're not allowed to have anything else named x. Once you've got a field called x, you cannot have another field, property, nested type, or event called x.

Thanks to overloading, methods are a bit of an oddity here. One way to characterize declaration spaces in the context of methods would be to say that "the set of all overloaded methods in a class that have the same name" constitutes an "entity". Another way to characterize methods would be to tweak the definition of declaration space to say that no two things are allowed to have the same name *except for a set of methods that all differ in signature*.

In short, scope answers the question "where can I use this name?" and declaration space answers the question "where is this name unique?"

Lifetime and scope are often confused because of the strong connection between the lifetime and scope of a local variable. The most succinct way to put it is that the contents of a local variable are guaranteed to be alive at least as long as the current "point of execution" is inside the scope of the local variable. "At least as long" of course implies "or longer"; capturing a local variable, for example, extends its lifetime.

