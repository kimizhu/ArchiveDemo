# Implementing the virtual method pattern in C\#, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/21/2011 7:08:00 AM

-----

(This is part two of a three-part series; part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/17/implementing-the-virtual-method-pattern-in-c-part-one.aspx); part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/24/implementing-the-virtual-method-pattern-in-c-part-three.aspx).)

So far we've gotten rid of instance methods; they're just static methods that take a hidden "this" parameter. But virtual methods are a bit harder. We're going to **implement virtual methods as fields of delegate type containing delegates to static methods.**

abstract class Animal  
{  
  public Func\<Animal, string\> Complain;  
  public Func\<Animal, string\> MakeNoise;  
  public static string MakeNoise(Animal \_this)  
  {  
    return "";  
  }  
}

OK, everything seems fine so far...

 

class Giraffe : Animal  
{  
  public bool SoreThroat { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return \_this.SoreThroat ? "What a pain in the neck\!" : "No complaints today.";  
  }  
}

Trouble. "Animal" does not have a property SoreThroat. But Complain cannot take a Giraffe, because then it is not compatible with the delegate type, which after all, expects an Animal as the "\_this" formal parameter.

What we need to make the virtual method pattern work is a guarantee that the caller will never pass in a Cat to a "virtual" method that is expecting a Giraffe. Let's assume that we have such a guarantee. Since we have that guarantee, we can make a conversion:

 

class Giraffe : Animal  
{  
  public bool SoreThroat { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Giraffe).SoreThroat ? "What a pain in the neck\!" : "No complaints today.";  
  }  
}  
class Cat : Animal  
{  
  public bool Hungry { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Cat).Hungry ? "GIVE ME THAT TUNA\!" : "I HATE YOU ALL\!";  
  }  
  public static string MakeNoise(Animal \_this)  
  {  
    return "MEOW MEOW MEOW MEOW MEOW MEOW";  
  }  
}  
class Dog : Animal  
{  
  public bool Small { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return "Our regressive state tax code is... SQUIRREL\!";  
  }  
  public static string MakeNoise(Dog \_this)  // Remember, we forgot to say "override"  
  {  
    return \_this.Small ? "yip" : "WOOF";  
  }  
}

Everything good? Not yet. We forgot to initialize the fields\!

Here for the first time we come to something that you actually cannot do in "C\# without instance methods". In the CLR, the virtual method "fields" are actually initialized **after the memory allocator runs but before the constructor runs**. (\*) We have no way in C\# of doing that. So let's go crazy here; we've already gotten rid of instance methods; instance constructors are basically just instance methods that are called when the object is created. So let's get rid of instance constructors too. Instead, instance constructors will be replaced by the factory pattern, where a static method creates and initializes the object. (We presume that this static method has the powers of a constructor; for example it is allowed to set readonly fields, and so on.) A call to a default constructor now does nothing but allocate memory.

 

abstract class Animal  
{  
  public Func\<Animal, string\> Complain;  
  public Func\<Animal, string\> MakeNoise;  
  public static string MakeNoise(Animal \_this)  
  {  
    return "";  
  }  
  // No factory; Animal is abstract.  
  public static void InitializeVirtualMethodFields(Animal animal)  
  {  
    animal.Complain = null; // abstract\!  
    animal.MakeNoise = Animal.MakeNoise;  
  }  
}  
class Giraffe : Animal  
{  
  public bool SoreThroat { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Giraffe).SoreThroat ? "What a pain in the neck\!" : "No complaints today.";  
  }  
  public static void InitializeVirtualMethodFields(Giraffe giraffe)  
  {  
    Animal.InitializeVirtualMethodFields(giraffe);  
    giraffe.Complain = Giraffe.Complain;  
    // Giraffe did not override MakeNoise, so ignore it.  
  }   
  public static Giraffe Create()  
  {  
    // There are no more instance constructors; this just allocates the memory.  
    Giraffe giraffe = new Giraffe();  
    // Ensure that virtual method fields are initialized before other code is run.  
    Giraffe.InitializeVirtualMethodFields(giraffe);  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}  
class Cat : Animal  
{  
  public bool Hungry { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return (\_this as Cat).Hungry ? "GIVE ME THAT TUNA\!" : "I HATE YOU ALL\!";  
  }  
  public static string MakeNoise(Animal \_this)  
  {  
    return "MEOW MEOW MEOW MEOW MEOW MEOW";  
  }  
  public static void InitializeVirtualMethodFields(Cat cat)  
  {  
    Animal.InitializeVirtualMethodFields(cat);  
    cat.Complain = Cat.Complain;  
    cat.MakeNoise = Cat.MakeNoise;  
  }   
  public static Cat Create()  
  {  
    Cat cat = new Cat();  
    Cat.InitializeVirtualMethodFields(cat);  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}  
class Dog : Animal  
{  
  public bool Small { get; set; }  
  public static string Complain(Animal \_this)  
  {  
    return "Our regressive state tax code is... SQUIRREL\!";  
  }  
  public static string MakeNoise(Dog \_this)  // Remember, we forgot to say "override"  
  {  
    return \_this.Small ? "yip" : "WOOF";  
  }  
  public static void InitializeVirtualMethodFields(Dog dog)  
  {  
    Animal.InitializeVirtualMethodFields(dog);  
    dog.Complain = Dog.Complain;  
    // Dog did not override MakeNoise, so ignore it.  
  }   
  public static Dog Create()  
  {  
    Dog dog = new Dog ();  
    Dog.InitializeVirtualMethodFields(dog);  
    // Now do the rest of the initialization that the constructor would have done.  
  }  
}

What about the call sites?  We rewrite our program as:

 

string s;  
Animal animal = Giraffe.Create();  
// creates a new Giraffe and initializes the Complain field to Giraffe.Complain,  
// and initializes the MakeNoise field to Animal.MakeNoise. We continue to  
// rewrite the calls so that the "receiver" is passed as "\_this" to each delegate:

s = animal.Complain(animal);  
// Invokes delegate animal.Complain, which refers to static method Giraffe.Complain  
  
s = animal.MakeNoise(animal);  
// invokes delegate animal.MakeNoise, which refers to static method Animal.MakeNoise  
  
animal = Cat.Create();  
// Creates a new Cat and initialzies the fields to Cat.Complain and Cat.MakeNoise.  
  
s = animal.Complain(animal);  // I hate you  
s = animal.MakeNoise(animal); // meow\!  
  
Dog dog = Dog.Create();  
// Initializes the fields to Dog.Complain and Animal.MakeNoise  
  
animal = dog;  
s = animal.Complain(animal);  
s = animal.MakeNoise(animal);  
// Invokes delegate animal.MakeNoise, which refers to static method Animal.MakeNoise  
  
s = Dog.MakeNoise(dog); // yip\!  
// Does not invoke a delegate at all; overload resolution sees that Dog has a valid  
// MakeNoise method that is declared on a more-derived type than the delegate  
// field of the base class, and chooses to call the more-derived static method.

And we're done; we've successfully emulated virtual and instance methods in a language that only has static methods (and delegates to static methods.)

However, **this is not very space-efficient**. Suppose there were a hundred virtual methods on Animal instead of two. That means that every class derived from Animal has a hundred fields, and in most of them, those fields are exactly the same, all the time. You make three hundred giraffes and each one of them will have exactly the same delegates in those hundred fields. This seems redundant and wasteful.

**Next time** we'll solve this memory wastage problem.

\-----

(\*) The rules of the CLR are that the virtual function "slots" are correctly initialized as soon as the object is created; this is in contrast to the rules of C++, which say that the values of the virtual functions change as the object goes through its construction process. Before I started on this team I was very much in favour of the C++ approach, as you can see from [this blog post from 2005](http://blogs.msdn.com/b/scottwil/archive/2005/01/14/353177.aspx) shortly before I joined the C\# team. Both approaches have their pros and cons; I now think the way the CLR does it is marginally better, but still it is best to not tempt fate: simply don't call virtual methods in constructors.

(This is part two of a three-part series; part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/17/implementing-the-virtual-method-pattern-in-c-part-one.aspx); part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/24/implementing-the-virtual-method-pattern-in-c-part-three.aspx).)

