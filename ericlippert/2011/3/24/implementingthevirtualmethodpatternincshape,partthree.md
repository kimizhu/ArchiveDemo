# Implementing the virtual method pattern in C\#, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/24/2011 7:10:00 AM

-----

(This is part three of a three-part series; part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/17/implementing-the-virtual-method-pattern-in-c-part-one.aspx); part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/21/implementing-the-virtual-method-pattern-in-c-part-two.aspx).)

Last time we saw how you could emulate virtual methods in a language that only had static methods by creating fields of delegate type, and then choosing what delegates go into the fields. However, this is not very space-efficient. Suppose there were a hundred virtual methods on Animal instead of two. That means that every class derived from Animal has a hundred fields, and in most of them, those fields are exactly the same, all the time. You make three hundred giraffes and each one of them will have *exactly the same delegates* in those hundred fields. This seems redundant and wasteful.

What the CLR actually does is it bundles those up into a virtual function dispatch table, or "vtable" for short. A vtable is a collection of delegates; the purpose of the vtable is to answer the question "if a virtual method was invoked on an object of this runtime type, which method should be called?"

 

sealed class VTable  
{  
  public readonly Func\<Animal, string\> Complain;  
  public readonly Func\<Animal, string\> MakeNoise;  
  public VTable(Func\<Animal, string\> complain, Func\<Animal, string\> makeNoise)  
  {  
    this.Complain = complain;  
    this.MakeNoise = makeNoise;  
  }  
}

Suppose we want to construct vtables for Giraffe, Cat and Dog to answer the questions "if these methods were invoked on an object of compile-time type Animal but runtime type Giraffe/Cat/Dog, which methods should actually be called?" That's easy enough:

 

static VTable GiraffeVTable = new VTable(Giraffe.Complain, Animal.MakeNoise);  
static VTable CatVTable = new VTable(Cat.Complain, Cat.MakeNoise);  
static VTable DogVTable = new VTable(Dog.Complain, Animal.MakeNoise);

And now we can just have every class have a reference to the appropriate vtable. We rewrite the classes again as follows:

 

abstract class Animal  
{  
  public VTable VTable;  
  public static string MakeNoise(Animal \_this)  
  {  
    return "";  
  }  
  // No factory; Animal is abstract.  
}  
class Giraffe : Animal  
{  
  private static VTable GiraffeVTable = new VTable(Giraffe.Complain, Animal.MakeNoise);  
  public bool SoreThroat { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Giraffe).SoreThroat ? "What a pain in the neck\!" : "No complaints today.";  
  }  
  public static Giraffe Construct()  
  {  
    // There are no more instance constructors; this just allocates the memory.  
    Giraffe giraffe = new Giraffe();  
    // Ensure that virtual method fields are initialized before other code is run.  
    giraffe.VTable = Giraffe.VTable;  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}  
class Cat : Animal  
{  
  private static VTable CatVTable = new VTable(Cat.Complain, Cat.MakeNoise);  
  public bool Hungry { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Cat).Hungry ? "GIVE ME THAT TUNA\!" : "I HATE YOU ALL\!";  
  }  
  public static string MakeNoise(Animal \_this)  
  {  
    return "MEOW MEOW MEOW MEOW MEOW MEOW";  
  }  
  public static Cat Construct()  
  {  
    Cat cat = new Cat();  
    cat.VTable = Cat.VTable;  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}  
class Dog : Animal  
{  
  private static VTable DogVTable = new VTable(Dog.Complain, Animal.MakeNoise);  
  public bool Small { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return "Our regressive state tax code is... SQUIRREL\!";  
  }  
  public static string MakeNoise(Dog \_this)  // Remember, we forgot to say "override"  
  {  
    return \_this.Small ? "yip" : "WOOF";  
  }  
  public static Dog Construct()  
  {  
    Dog dog = new Dog();  
    dog.VTable = Dog.VTable;  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}

And now we have to rewrite the call sites again:

 

string s;  
Animal animal = Giraffe.Create();  
s = animal.VTable.Complain(animal);  
s = animal.VTable.MakeNoise(animal);  
animal = Cat.Create();  
s = animal.VTable.Complain(animal);  
s = animal.VTable.MakeNoise(animal);  
Dog dog = Dog.Create();  
animal = dog;  
s = animal.VTable.Complain();  
s = animal.VTable.MakeNoise();  
s = Dog.MakeNoise(dog);  

That is basically how the virtual method pattern is implemented behind the scenes. The C\# compiler analyzes every method that is declared as abstract, virtual or override and figures out which "vtable slot" that method corresponds to; it then emits metadata that has enough information to allow the runtime to lay out a vtable. For every call site, the C\# compiler works out whether to call an instance method directly or do an indirection through the vtable, and emits the code accordingly. When the object is allocated at runtime, the CLR consults the metadata to figure out which vtable to plug into the hidden vtable field of each object. The vtable is of course not actually a class, and its fields are not actually delegates; the CLR has a less heavyweight way to represent a reference to a method internally. But the concept is the same.

Obviously I've omitted a great many details here. For example, it is not legal to invoke a virtual or instance method with a null receiver, but it is legal for a static method to be called with a null first argument. By rights, when rewriting the call sites to go from instance or virtual calls to static calls, I should have been putting in null checks as well. I've also not at all described how "base" calls work (though perhaps you can figure it out from this sketch). I've also not said how this works for boxed value types that override virtual methods of object, though again, perhaps you can figure it out. I've ignored issues of security and accessibility. I've omitted describing what happens when a class creates a "new virtual" method. And I've said nothing about how interfaces work; the CLR uses special techniques to solve interface overriding problems that I might discuss in a later blog.

The real takeaway here is that by choosing to embed the "virtual and instance calls" pattern in the language, you get to take advantage of its power without having to go through all the pain of declaring those fields and setting up those vtables yourself. The compiler and CLR teams get to experience that pain for you; you're welcome.

You also get safety\! The pattern as implemented here depends on everyone following the rules of the pattern carefully. For instance, what stops you from calling giraffe.VTable.Complain(dog)?  Nothing, that's what. The C\# compiler ensures that it is impossible to actually call a virtual method with a "this" that does not match the type expected by the function, but in our pattern-based implementation of virtual methods that error is not trapped until the conversion fails at runtime, if it is caught at all.

(This is part three of a three-part series; part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/17/implementing-the-virtual-method-pattern-in-c-part-one.aspx); part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/21/implementing-the-virtual-method-pattern-in-c-part-two.aspx).)

