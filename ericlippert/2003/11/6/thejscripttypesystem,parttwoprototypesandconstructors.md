# The JScript Type System, Part Two: Prototypes and constructors

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/6/2003 5:01:00 PM

-----

A number of readers made some good comments on my article on JScript typing that deserve to be called out in more detail.

 

First, I was being a little sloppy in my terminology -- I casually conflated **static typing** with **strong typing**, and **dynamic typing** with **weak typing**.  Thanks for calling me on that.  Under the definitions proposed by the reader, JScript would be a dynamically typed language (because every variable can take a value of any type) and a strongly typed language (because every object knows what type it is.)  By contrast, C++ is a statically typed language (because every variable must have a type, which the compiler enforces) but also a weakly typed language (because the reinterpret cast allows one to turn pointers into integers, and so on.)

 

Second, a reader notes that one of the shortcomings of JScript is that though it is a strongly typed language (in our new sense) that **it is a royal pain to actually determine the runtime type an object**.  The typeof operator has a number of problems:

 

\* null is listed as being of the object type, though technically it is a member of the Null type. 

\* primitives (strings, numbers, Booleans) wrapped in objects are listed as being of the object type rather than their underlying type.

\* JScript, unlike VBScript, does not interrogate COM objects to determine the class name.

\* If JScript is passed a variant from the outside world that it cannot make sense of then typeof returns "unknown".

 

Perhaps there is some other way.  Prototype inheritance affords a kind of type checking, for example.

 

Prototype inheritance works like this.  Every JScript object has an object (or possibly null) called its **prototype object**.  So suppose an object foo has prototype object bar, and bar has prototype object baz, and baz has prototype object null.  If you call a method on foo then JScript will search foo, bar and baz for that method, and call the first one it finds.  The idea is that one object is a prototypical object, and then other objects specialize it.  This allows for code re-use without losing the ability to dynamically customize behaviour of individual objects.

 

Prototypes are usually done something like this:

 

var Animal = new Object(); 

// omitted: set up Animal object 

function Giraffe(){ 

// omitted: initialize giraffe object. 

} 

Giraffe.prototype = Animal; 

var Jerry = new Giraffe(); 

 

Now Jerry has all the properties and methods of an individual Giraffe object AND all the properties and methods of Animal.  You can use IsPrototypeOf to see if a given object has Animal on its prototype chain.  Since prototype chains are immutable once created, this gives you a pretty reliable sort of type checking.  

 

Note that Giraffe is not a prototype of Jerry.  Note also that Animal is not the prototype of Giraffe\!  **The object which is assigned to the** **prototype *property* of the constructor is the prototype of the instance.**

  

Now, you guys are not the first people to point out to me that determining types is tricky.  A few years ago someone asked me what the differences are amongst 

 

if (func.prototype.IsPrototypeOf(instance))

 

and

 

if (instance.constructor == func)

 

and 

 

if (instance instanceof func)

 

The obvious difference is that the first one looks at the **whole prototype chain**, whereas the second two look at **the** **constructor**, right? Or is that true?  Is there a semantic difference between the last two?  Actually, there is. Let's look at some examples, starting with one that seems to show that there is no difference: 

 

function Car(){}

var honda = new Car();

print(honda instanceof Car); // true

print(honda.constructor == Car);  // true

 

It appears that instance instanceof func and instance.constructor == func have the same semantics.   *They do not.*  Here's a more complicated example that demonstrates the difference:

 

var Animal = new Object();

function Reptile(){ }

Reptile.prototype = Animal;

var lizard = new Reptile();

print(lizard instanceof Reptile); // true

print(lizard.constructor == Reptile); // false

 

In fact lizard.constructor is equal to Object, not Reptile.

 

Let me repeat what I said above, because no one understands this the first time -- I didn't, and I've found plenty of Javascript books that get it wrong.  When we say

 

Reptile.prototype = Animal;

 

this does NOT mean "the prototype of Reptile is Animal".  It cannot mean that because (obviously\!) the prototype of Reptile, a function object, is Function.prototype.  No, this means "**the prototype of any instance of** Reptile** is **Animal".  *There is no way to directly manipulate or read the prototype chain of an existing object.*

 

Now that we've got that out of the way, the simple one first:

 

instance instanceof func means "is the prototype property of func equal to any object on instance's prototype chain?"  So in our second example, the prototype property of Reptile is Animal and Animal is on lizard's prototype chain.  

 

But what about our first example where there was no explicit assignment to the Car prototype?

 

The compiler creates a function object called "Car".  It **also creates a default prototype object and assigns it to** Car.prototype.  So again, when we way 

 

print(honda instanceof Car);

 

the instanceof operator gets the prototype property (Car.prototype) and compares it to the prototype chain of honda.  Since honda was constructed by Car it gets Car.prototype on its prototype chain.

 

To sum up the story so far,

