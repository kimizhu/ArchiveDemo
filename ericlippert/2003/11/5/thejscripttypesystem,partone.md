# The JScript Type System, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/5/2003 1:38:00 PM

-----

I thought I might spend a few days talking about the JScript and JScript .NET type systems, starting with some introductory material.

Consider a JScript variable:

 

var myVar;

Now think about the possible values you could store in the variable. A variable may contain any number, any string or any object. It can also be true or false or null or even undefined. This is a rather large set of possible values. In fact, the set of all legal values is infinite.Countably infinite, and in practice limited by available memory, but in theory there is no upper limit.

A type**** is characterized by two things, a **set** and a **rule**. First, **a type consists of a subset (possibly infinitely large) of the set of all possible values**. Second, a type defines **a rule for transforming values outside the set into values in the set**. (This rule may specify that certain values are not convertible and hence produce "type mismatch" errors.)

For example, **String** is a type. The set of all possible strings is an (infinite) subset of the set of all possible values, and there are rules for determining how all non-string values are converted into strings.

JScript Classic is a **dynamically typed language**. This means that any value of any type may be assigned to any variable without restriction. It is often said -- inaccurately -- that "JScript has only one type". This is true only in the sense that JScript has no *restrictions* on what data may be assigned to any variable, and in that sense every variable is "the same type" – namely, the "any possible value" type. However, the statement is misleading because it implies that JScript supports no types at all, when in fact it supports six built-in types.

JScript .NET, by contrast, is an **optionally statically-typed language**. A JScript .NET variable may be given a type annotation which restricts the values which may be stored in the variable. This annotation is optional; an unannotated variable acts like a JScript variable and may be assigned any value.

JScript has the property that a value can always describe its own type at runtime. This is not true in, say, C, where you can have a void\* and no way of asking it "are you pointing to an integer or a string?" In JScript, you can always ask a value what its type is and it will tell you.

The concept of **subtyping** is not particularly important in JScript Classic though it will become quite useful when we discuss JScript .NET classes later. Essentially a type T1 is a subtype of another type T2 if T1's set of values is a subset of T2's set of values. A type consisting of the set of all integers might be a subtype of a type consisting of all the numbers, for instance. (This is not how the integers are traditionally construed; the C type system makes integers and floats disjoint types, where the integer 1 and the float 1.0 are different values that happen to compare as equal -- but comparisons across types is a subject for a later blog entry.)

Anyway, JScript Classic has six built-in types, all of which are disjoint. They are as follows:

The **Number** type contains all floating-point numbers as well as positive Infinity, negative Infinity and a special Not-a-Number ("NaN") values. It may seem odd that "Not-a-Number" is a Number but this does in fact make sense. NaN is the value returned when an operation logically must return a number but no actual number makes sense. For example, when trying to convert the string "banana" to a Number, NaN is the result. Because numbers in JScript are actually represented by a 64 bit floating point number there are a finite number of possible Number values. The number of numbers is very large (in fact there are 18437736874454810627 possible numbers, which is just shy of 2^64.) Numbers have approximately fifteen decimal digits of precision and can range from as tiny as 2.2 x 10^-308 to as large as 1.7 x 10^308.

The **String** type contains all Unicode strings of any length (including zero-length empty strings.) The string type is for all practical purposes infinite, as the length of a string is limited only by the ability of the operating system to allocate enough memory to hold it.

The **Boolean** type has two values: true and false

The **Null** type has one value: null

The **Undefined** type has one value: undefined. All uninitialized JScript variables are automatically set to undefined

The **Object** type has an infinite number of values. An object is essentially a collection of named properties where each property can be a value of any type. In JScript many things are objects: functions, dates, arrays and regular expressions are all objects.

Types are not themselves "first class" objects in JScript, though they are in JScript .NET. I'll discuss that, along with the differences between prototype and class inheritance, in later entries.

