# Covariance and Contravariance in C\#, Part Three: Method Group Conversion Variance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/19/2007 10:13:00 AM

-----

Last time I discussed how array covariance is broken in C\# (and Java, and a number of other languages as well.) Today, a non-broken kind of variance supported by C\# 2.0: conversions from method groups to delegates. This is a more complicated kind of variance, so let me spell it out in more detail.

Suppose that you have a method which returns a Giraffe:

 

static Giraffe MakeGiraffe() { …

Suppose further that you have a delegate type representing a function which takes no arguments and returns an Animal. Say, Func\<Animal\>. Should this implicit conversion from method group to delegate be legal?

 

Func\<Animal\> func = MakeGiraffe;

The caller of func is expecting an Animal to be returned. The actual function captured by the delegate always returns a Giraffe, which is an Animal, so the caller of func is never going to get anything that they’re not capable of dealing with. There is no problem in the type system here. Therefore *we can make method group to delegate conversions **covariant*** (‡) ***in their return types**.*

Now suppose you have two methods, one which takes a Giraffe and one which takes an Animal:

 

void Foo(Giraffe g) {}  
void Bar(Animal a) {}

and a delegate to a void-returning function that takes a Mammal:

 

Action\<Mammal\> action1 = Foo; // illegal  
Action\<Mammal\> action2 = Bar; // legal

Why is the first assignment illegal? Because the caller of action1 can pass a Tiger, but Foo cannot take a Tiger, only a Giraffe\! The second assignment is legal because Bar can take any Animal.

In our previous example we preserved the direction of the assignability: Giraffe is smaller than Animal, so a method which returns a Giraffe is smaller than a delegate which returns an Animal. In this example we are reversing the direction of the assignability: Mammal is smaller than Animal, so a method which takes an Animal is smaller than a delegate which takes a Mammal. Because the direction is reversed, *method group to delegate conversions are **contravariant** **in their argument types***.

Note that all of the above applies only to *reference* types. We never say something like “well, every int fits into a long, so a method which returns an int is assignable to a variable of type Func\<long\>”.

Next time: a stronger kind of delegate variance that we could support in a hypothetical future version of C\#.

(‡) A note to nitpickers out there: yes, I said earlier that variance was a property of *operations on types*, and here I have an operation on *method groups*, which are *typeless* expressions in C\#. I’m writing a blog, not a dissertation; deal with it\!

