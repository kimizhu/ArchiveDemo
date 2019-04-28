# Following the pattern

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/30/2011 9:04:39 AM

-----

Here's a question I got from a user recently:

**The foreach loop in C\# uses a pattern-based approach to semantic analysis. LINQ in C\# also uses a pattern-based approach. Why don't other features, such as the "using" statement, also use a pattern-based approach?**

What a great question\!

First off, what do we mean by a "pattern-based approach"?

You might think that in order to use a foreach loop, the collection you are iterating over must implement IEnumerable or IEnumerable\<T\>. But as it turns out, that is not actually a requirement. What is required is that the type of the collection must have a public method called GetEnumerator, and that must return some type that has a public property getter called Current and a public method MoveNext that returns a bool. If the compiler can determine that all of those requirements are met then the code is generated to use those methods. Only if those requirements are *not* met do we check to see if the object implements IEnumerable or IEnumerable\<T\>.

Essentially, C\# is doing a fairly strong form of "duck typing" here. That is, if the object is known at compile time to have a Quack() method then we treat it like a duck even if it isn't marked as implementing IDuck. There are a few places in the C\# language where we do this sort of "pattern matching"; we don't care what the exact type is, just so long as the methods we need are available.

But why do we use a pattern-based approach in foreach in the first place?

Two reasons. First, imagine a world without generic types: the world of C\# 1.0. Suppose you have a collection that you yourself have just implemented, a collection of ten square integers:

class MyIntegers : IEnumerable  
{  
  private class MyEnumerator : IEnumerator  
  {  
    private int index = 0;  
    public object Current { return index \* index; }  
    public bool MoveNext()  
    {  
      if (index \> 10) return false;  
      ++index;  
      return true;  
    }  
  }  
  public IEnumerator GetEnumerator() { return new MyEnumerator(); }  
}

  
When you "foreach" over this thing, every integer that comes out of the collection is boxed. How can you avoid the boxing penalty? Like this:

class MyIntegers : IEnumerable  
{  
  public class MyEnumerator : IEnumerator  
  {  
    private int index = 0;  
    object IEnumerator.Current { return this.Current; }  
    int Current { return index \* index; }  
    public bool MoveNext()  
    {  
      if (index \> 10) return false;  
      ++index;  
      return true;  
    }  
  }  
  public MyEnumerator GetEnumerator() { return new MyEnumerator(); }  
  IEnumerator IEnumerable.GetEnumerator() { return this.GetEnumerator(); }  
}

  
Now the foreach loop sees can use the public method GetEnumerator that returns a MyEnumerator, that in turn has a public property Current that returns an int. No boxing required.

The second reason is because using this approach, the "MyEnumerator" could also be a mutable struct. Yes, mutable structs are a bad programming practice, but in this case it's not so bad because the "foreach" loop is never going to expose the raw enumerator to possible misuse by user code. Sometimes you really do want to avoid collection pressure, so making the enumerator a struct can be a small but measurable win. (As always, do not make things into structs "because they are faster" unless you have empirical evidence that doing so makes a measurable and important difference.)

So there you go: we use a pattern-based approach for "foreach" because first, we didn't have the ability to avoid boxing via IEnumerable\<T\> in C\# 1.0, and because there is a small but perhaps important performance win in avoiding the pressure of allocating a reference-typed enumerator.

What about LINQ? We also use a pattern-based approach in LINQ. When you say "**from c in customers where ...**" we do not require that "customers" have a "Where" method, or even that it implement IEnumerable. We don't have any type requirements on it at all; rather, **we require that overload resolution finds an instance method or an extension method called "Where" that works with the LINQ pattern.**

Again, two reasons. The first reason is that LINQ was invented very late; it would not be a success if we required everyone who wanted to have their data source work with LINQ have to ship a new version of their interface that has Where and Select methods. It has to be a feature that "lights up" without any work on behalf of the data source providers.

The second and more highfalutin reason is that LINQ was designed to bake the "monad" pattern into C\#. It was specifically designed to bake in the "sequence" monad, but if you are a crazy person you can actually use query comprehension syntax to represent binding operations on any monad, [as Wes points out in his excellent article](http://blogs.msdn.com/b/wesdyer/archive/2008/01/11/the-marvels-of-monads.aspx). People sometimes ask why there is no "IMonad" interface in C\# that identifies a type that can be used as a monad. The short answer is that the concept of "I am a type that obeys the rules of a monad" is actually too general for the .NET type system to handle; the monad pattern is a "higher type" than an interface. Therefore, rather than bake it into the type system, we baked it into the compiler. (\*)

So now we come to the user's actual question: given that we use this pattern-based approach in several places in C\#, why not use it everywhere? Why require that an object implement IDisposable to be disposed in a "using" block? Why not just require that it implement a public method called Dispose?

The answer is that we *like* capturing things in type systems; we do so whenever we can. Types are our first and best solution for solving problems in a statically-analyzed language, and so we want to stick to that whenever possible. In the case of the "using" block there is no compelling reason to not require that a disposable object mark itself with a special interface. [We've already seen that we can elide boxing in things that get disposed](http://blogs.msdn.com/b/ericlippert/archive/2011/03/14/to-box-or-not-to-box-that-is-the-question.aspx), and even if we cannot, the boxing penalty is pretty small for disposing a dozen handle-holding objects compared to the penalty of boxing a million integers coming out of a collection. And the LINQ reasoning doesn't apply to "using" -- we do not want you to be able to extend the disposal behaviour of objects you do not own, even as we do want you to be able to extend the sorting, searching and grouping behaviours of objects you do not own. With no compelling reason for a pattern-based approach, and no compelling reason against an interface-based approach, we went with the interface.

However, as the C\# language gets more and more high-level features added to it, we are finding that often a pattern-based approach is necessary. The coming "async/await" feature, for example, will use a LINQ-like pattern-based approach to figuring out how to make an "awaiter" for a particular "task" object. Pattern-based approaches can allow more participation by third parties and enable language designers to embed design patterns in the language that are too rich for their type systems to handle, so I would expect to see more of this sort of thing in the future.

\--------------------

(\*) A detailed explanation of why the .NET type system is insufficient to represent higher types would take us pretty far afield. If this subject interests you, [see this StackOverflow question](http://stackoverflow.com/questions/1709897). For more ideas on how the generic type system of .NET is insufficient to represent certain desired patterns, [see my recent article on the subject](http://blogs.msdn.com/b/ericlippert/archive/2011/02/03/curiouser-and-curiouser.aspx). If you want an even more gentle introduction to monads than Wes's article, see [this StackOverflow question](http://stackoverflow.com/questions/2704652).

