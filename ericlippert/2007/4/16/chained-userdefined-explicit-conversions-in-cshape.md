<div id="page">

# Chained user-defined explicit conversions in C\#

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/16/2007 10:00:00 AM

-----

<div id="content">

<div class="mine">

Reader Niall asked me why the following code compiles but produces an exception at runtime:

<span class="code"> </span>

class Base {}  
class Derived : Base {}  
class Castable {  
  public static explicit operator Base() {  
    return new Base();  
  }  
}  
// ...  
Derived d = (Derived)(new Castable());

It should be clear why this produces an exception at runtime; the user-defined operator returns a <span class="code">Base</span> and that is not assignable to a variable of type <span class="code">Derived</span>. But why does the compiler allow it in the first place?

First off, let’s define the difference between an implicit and an explicit conversion. An implicit conversion is one which the compiler knows can always be done without incurring the risk of a runtime exception. When you have a method <span class="code">int Foo(int i){...}</span> and call it with <span class="code">long l = Foo(myshort);</span>, the compiler inserts implicit conversions from <span class="code">int</span> to <span class="code">long </span>on the return side and from <span class="code">short</span> to <span class="code">int</span> on the call side. There is no <span class="code">int </span>which doesn’t fit into a <span class="code">long </span>and there is no <span class="code">short</span> which doesn’t fit into an <span class="code">int</span>, so we know that the conversions will always succeed, so we just up and do them for you.

There are also conversions which we know at compile time will never succeed. If there is no user-defined conversion from <span class="code">Giraffe</span> to <span class="code">int</span>, then <span class="code">Foo(new Giraffe())</span> is always going to fail at runtime, so this fails at compile time.

An explicit conversion is a conversion which might succeed sometimes but might also fail. We cannot disallow it, because it might succeed, but we can’t go silently inserting one either, since it might fail unexpectedly. We need to force the developer to acknowledge that risk explicitly. If you called <span class="code">ulong ul = Foo(mynullableint);</span> then that might fail, so the compiler requires you to spell out that the conversions are explicit. The assignment could be written <span class="code">ulong ul = (ulong)Foo((int)mynullableint);</span>.

There are two times that the compiler will insert an explicit cast for you without producing a warning or error. The first is the case above. When a user-defined explicit cast requires an explicit conversion on either the call side or the return side, the compiler will insert the explicit conversions as needed. The compiler figures that if the developer put the explicit cast in the code in the first place then the developer knew what they were doing and took the risk that any of the conversions might fail. That’s what the cast means: this conversion might fail, I will deal with it.

I understand that this puts a burden upon the developer to fully understand the implications of a cast, but the alternative is to make you spell it out even further, and it just gets to be too much. The logical extreme of this would be a case such as

<span class="code"> </span>

public struct S{  
    public static explicit operator decimal?(S s) {return 1.0m;}  
}  
//...  
S? s = new S();  
int i = (int) s;

Here we do first an explicit conversion from <span class="code">S?</span> to <span class="code">S</span>, then a user-defined explicit conversion from <span class="code">S</span> to <span class="code">decimal?</span>, then an explicit conversion from <span class="code">decimal?</span> to <span class="code">decimal</span>, and then an explicit conversion from <span class="code">decimal</span> to <span class="code">int</span>. That’s four explicit conversions for the price of one cast, which I think is pretty good value for your money.

I want to note at this point that this is as long as the chain gets. A user-defined conversion can have built-in conversions inserted automatically on the call and return sides, but we never automatically insert other user-defined conversions. We never say that there’s a user-defined conversion from <span class="code">Alpha</span> to <span class="code">Bravo</span>, and a user-defined conversion from <span class="code">Bravo</span> to <span class="code">Charlie</span>, and therefore casting an <span class="code">Alpha</span> to a <span class="code">Charlie</span> is legal. That doesn’t fly.

And a built-in conversion can be lifted to nullable, which may introduce additional conversions as in the case above. But again, these are never user-defined.

The second is in the <span class="code">foreach</span> loop. If you have <span class="code">foreach(Giraffe g in myAnimals)</span> then we generate code which fetches each member of the collection and does an “explicit” conversion to <span class="code">Giraffe</span>. If there happens to be a <span class="code">Snail</span> or a <span class="code">WaterBuffalo</span> in <span class="code">myAnimals</span>, that’s a runtime exception. I considered adding a warning to the compiler for this case to say hey, be aware that your collection is of a more general type, this could fail at runtime. It turns out that there are so many programs which use this programming style, and so many of them have “compile with warnings as errors” turned on in their builds, that this would be a huge breaking change. So we opted to not do it.

</div>

</div>

</div>

