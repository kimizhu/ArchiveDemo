<div id="page">

# Covariance and Contravariance in C\#, Part Six: Interface Variance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/26/2007 10:30:00 AM

-----

<div id="content">

<div class="mine">

Over the last few posts I’ve discussed how it is possible to treat a delegate as contravariant in its arguments and covariant in its return type. A delegate is basically just an object which represents a *single* function call; we can do this same kind of thing to other things which represent function calls. Interfaces, for example, are *contracts which specify what set of function calls are available on a particular object*.

This means that we could extend the notion of variance to interface definitions as well, using the same rules as we had for delegates. For example, consider

<span class="code"> </span>

public interface IEnumerator\<T\> : IDisposable, IEnumerator {  
  new T Current { get; }  
}

Here we have a generic interface where the sole use of the parameter is in an output position. We could therefore make the parameter covariant. That would mean that it would be legal to assign an object implementing <span class="code">IEnumerator\<Giraffe\></span> to a variable of type <span class="code">IEnumerator\<Animal\></span>. Since the user of that variable will always expect an <span class="code">Animal</span> to come out, and the actual backing implementation always produces a <span class="code">Giraffe</span>, everyone is happy.

Once we’ve got <span class="code">IEnumerator\<+T\></span>, we can then notice that <span class="code">IEnumerable\<T\></span> is defined as:

<span class="code"> </span>

public interface IEnumerable\<T\> : IEnumerable {  
  new IEnumerator\<T\> GetEnumerator();  
}

Again, the parameter appears only in an output position, so we could make <span class="code">IEnumerable\<+T\></span> covariant as well.

This then opens up a whole slew of nice scenarios. Today, this code would fail to compile:

<span class="code"> </span>

void FeedAnimals(IEnumerable\<Animal\> animals) {  
  foreach(Animal animal in animals)  
    if (animal.Hungry)  
      Feed(animal);  
}  
...  
IEnumerable\<Giraffe\> adultGiraffes = from g in giraffes where g.Age \> 5 select g;  
FeedAnimals(adultGiraffes);

Because <span class="code">adultGiraffes</span> implements <span class="code">IEnumerable\<Giraffe\></span>, not <span class="code">IEnumerable\<Animal\></span>. With C\# 3.0 you’d have to do a silly and expensive casting operation to make this compile, something like:

<span class="code"> </span>

FeedAnimals(adultGiraffes.Cast\<Animal\>());

or

<span class="code"> </span>

FeedAnimals(from g in adultGiraffes select (Animal)g);

Or whatever. This explicit typing should not be necessary. Unlike arrays (which are read-write) it is perfectly typesafe to treat a read-only list of giraffes as a list of animals.

Similarly, we could make

<span class="code"> </span>

public interface IComparer\<-T\> {  
  int Compare(T x, T y);  
}

into a contravariant interface, since the type parameter is used only in *input* positions. You could then implement an object which compares two <span class="code">Animal</span>s and use it in a context where you need an object which compares two <span class="code">Giraffe</span>s without worrying about type system problems.

Next time: Suppose we were to do interface and delegate variance in a hypothetical future version of C\#. What would the syntax look like? Is this goofy plus and minus really the best we can do? Do we need any syntax at all?

</div>

</div>

</div>

