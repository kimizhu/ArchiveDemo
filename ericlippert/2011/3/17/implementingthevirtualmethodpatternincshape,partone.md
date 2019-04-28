# Implementing the virtual method pattern in C\#, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/17/2011 6:42:00 AM

-----

(This is part one of a three-part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/21/implementing-the-virtual-method-pattern-in-c-part-two.aspx).)

If you've been in this business for any length of time you've undoubtedly seen some of the vast literature on "design patterns" -- you know, those standard solutions to common problems with names like "factory" and "observer" and "singleton" and "iterator" and "composite" and "adaptor" and "decorator" and... and so on. It is frequently useful to be able to take advantage of the analysis and design skills of others who have already given considerable thought to codifying patterns that solve common problems. However, I think it is valuable to realize that *everything* in high-level programming is a design pattern. Some of those patterns are so good, we've baked them right into the language so thoroughly that most of us don't even think of them as examples patterns anymore, patterns like "type" and "function" and "local variable" and "call stack" and "inheritance".

I was asked recently how virtual methods work "behind the scenes": how does the CLR know at runtime which derived class method to call when a virtual method is invoked on a variable typed as the base class? Clearly it must have something upon which to make a decision, but how does it do so efficiently? I thought I might explore that question by considering **how you might implement the "virtual and instance method pattern" in a language which did not have virtual or instance methods**. So, for the rest of this series I am banishing virtual and instance methods from C\#. I'm leaving delegates in, but delegates can only be to static methods. Our goal is to take a program written in regular C\# and see how it can be transformed into C\#-without-instance-methods. Along the way we'll get some insights into how virtual methods really work behind the scenes.

Let's start with some classes with a variety of behaviours:

 

abstract class Animal  
{  
  public abstract string Complain();  
  public virtual string MakeNoise()  
  {  
    return "";  
  }  
}  
class Giraffe : Animal  
{  
  public bool SoreThroat { get; set; }  
  public override string Complain()  
  {  
    return SoreThroat ? "What a pain in the neck\!" : "No complaints today.";  
  }  
}  
class Cat : Animal  
{  
  public bool Hungry { get; set; }  
  public override string Complain()  
  {  
    return Hungry ? "GIVE ME THAT TUNA\!" : "I HATE YOU ALL\!";  
  }  
  public override string MakeNoise()  
  {  
    return "MEOW MEOW MEOW MEOW MEOW MEOW";  
  }  
}  
class Dog : Animal  
{  
  public bool Small { get; set; }  
  public override string Complain()  
  {  
    return "Our regressive state tax code is... SQUIRREL\!";  
  }  
  public string MakeNoise()  // We forgot to say "override"\!  
  {  
    return Small ? "yip" : "WOOF";  
  }  
}

Anyone who has spent five minutes in the same room as my cat and a can of tuna will recognize her influence on the program above.

OK, so we've got some abstract methods, some virtual base class methods, classes which do and do not override various methods, and one (accidental) instance method. We might have this program fragment:

 

string s;  
Animal animal = new Giraffe();  
s = animal.Complain();  // no complaints  
s = animal.MakeNoise(); // no noise  
animal = new Cat();  
s = animal.Complain();  // I hate you  
s = animal.MakeNoise(); // meow\!  
Dog dog = new Dog();  
animal = dog;  
s = animal.Complain();  // squirrel\!  
s = animal.MakeNoise(); // no noise  
s = dog.MakeNoise();    // yip\!

What has to happen here? Two interesting things. First, when Complain or MakeNoise is called on a value of type Animal, the call must be dispatched to the appropriate method based on the runtime type of the receiver. Second, when MakeNoise is called on a dog, somehow we have to do **one thing if the value was of *compile-time* type Dog, and a different thing if the value was of *compile-time* type Animal but *runtime* type Dog.**

How would we do this in a language without virtual or instance methods? Remember, every method has to be a static method.

Let's look at the non-virtual instance method first. That's straightforward. The callee can be written as:

 

public static string MakeNoise(Dog \_this)   
{  
  return \_this.Small ? "yip" : "WOOF";  
}

and the caller can be written as: 

s = Dog.MakeNoise(dog); // yip\!

The "instance method" pattern is easy: instance methods are nothing more than static methods that have an invisible "this" parameter. You just follow the pattern of always having the first formal parameter be called "\_this" and be of the current type, and you're done.

Virtual methods are bit harder. Somehow we have to figure out at runtime which one to dispatch.

By assumption we don't have virtual methods (and we can assume that we don't have virtual properties; they're just virtual methods in fancy dress.)

We do however have fields of delegate type. What if we:

(1) transform all the virtual and override methods into static methods that take an Animal as "this", as before, and  
(2) make delegate-typed fields of those methods, and then  
(3) transform the calls to invoke the delegates

? If we do that then we can choose which delegates to put in the fields of each instance, and thereby control which method is invoked.

**Next time** we'll give that a try.

(This is part one of a three-part series; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/03/21/implementing-the-virtual-method-pattern-in-c-part-two.aspx).)

