# Chained user-defined explicit conversions in C\#

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/16/2007 10:00:00 AM

-----

Reader Niall asked me why the following code compiles but produces an exception at runtime:

 

class Base {}  
class Derived : Base {}  
class Castable {  
  public static explicit operator Base() {  
    return new Base();  
  }  
}  
// ...  
Derived d = (Derived)(new Castable());

It should be clear why this produces an exception at runtime; the user-defined operator returns a Base and that is not assignable to a variable of type Derived. But why does the compiler allow it in the first place?

First off, let’s define the difference between an implicit and an explicit conversion. An implicit conversion is one which the compiler knows can always be done without incurring the risk of a runtime exception. When you have a method int Foo(int i){...} and call it with long l = Foo(myshort);, the compiler inserts implicit conversions from int to long on the return side and from short to int on the call side. There is no int which doesn’t fit into a long and there is no short which doesn’t fit into an int, so we know that the conversions will always succeed, so we just up and do them for you.

There are also conversions which we know at compile time will never succeed. If there is no user-defined conversion from Giraffe to int, then Foo(new Giraffe()) is always going to fail at runtime, so this fails at compile time.

An explicit conversion is a conversion which might succeed sometimes but might also fail. We cannot disallow it, because it might succeed, but we can’t go silently inserting one either, since it might fail unexpectedly. We need to force the developer to acknowledge that risk explicitly. If you called ulong ul = Foo(mynullableint); then that might fail, so the compiler requires you to spell out that the conversions are explicit. The assignment could be written ulong ul = (ulong)Foo((int)mynullableint);.

There are two times that the compiler will insert an explicit cast for you without producing a warning or error. The first is the case above. When a user-defined explicit cast requires an explicit conversion on either the call side or the return side, the compiler will insert the explicit conversions as needed. The compiler figures that if the developer put the explicit cast in the code in the first place then the developer knew what they were doing and took the risk that any of the conversions might fail. That’s what the cast means: this conversion might fail, I will deal with it.

I understand that this puts a burden upon the developer to fully understand the implications of a cast, but the alternative is to make you spell it out even further, and it just gets to be too much. The logical extreme of this would be a case such as

 

public struct S{  
    public static explicit operator decimal?(S s) {return 1.0m;}  
}  
//...  
S? s = new S();  
int i = (int) s;

Here we do first an explicit conversion from S? to S, then a user-defined explicit conversion from S to decimal?, then an explicit conversion from decimal? to decimal, and then an explicit conversion from decimal to int. That’s four explicit conversions for the price of one cast, which I think is pretty good value for your money.

I want to note at this point that this is as long as the chain gets. A user-defined conversion can have built-in conversions inserted automatically on the call and return sides, but we never automatically insert other user-defined conversions. We never say that there’s a user-defined conversion from Alpha to Bravo, and a user-defined conversion from Bravo to Charlie, and therefore casting an Alpha to a Charlie is legal. That doesn’t fly.

And a built-in conversion can be lifted to nullable, which may introduce additional conversions as in the case above. But again, these are never user-defined.

The second is in the foreach loop. If you have foreach(Giraffe g in myAnimals) then we generate code which fetches each member of the collection and does an “explicit” conversion to Giraffe. If there happens to be a Snail or a WaterBuffalo in myAnimals, that’s a runtime exception. I considered adding a warning to the compiler for this case to say hey, be aware that your collection is of a more general type, this could fail at runtime. It turns out that there are so many programs which use this programming style, and so many of them have “compile with warnings as errors” turned on in their builds, that this would be a huge breaking change. So we opted to not do it.

