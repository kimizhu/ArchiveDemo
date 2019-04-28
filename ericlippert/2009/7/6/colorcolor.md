# Color Color

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/6/2009 9:29:00 AM

-----

Pop quiz: What does the following code do when compiled and run?

 

class C  
{  
    public static void M(string x)  
    {  
        System.Console.WriteLine("static M(string)");  
    }  
    public void M(object s)  
    {  
        System.Console.WriteLine("M(object)");  
    }  
}  
class Program  
{  
    static void Main()  
    {  
        C c = new C();  
        c.M("hello");  
    }  
}

(1) writes static M(string)  
(2) writes M(object)  
(3) uh, dude, this code doesn’t even compile much less run  
(4) something else Think about that for a bit and then try it and see if you were right. . . . . . . . In option (1), the compiler could decide that the best match is the static one and let you call it through the instance, ignoring the instance value. In option (2), the compiler could decide that the object version is the best match and ignore the static one, even though the argument match is better. But neither of those actually happens; the rules of C\# say that the compiler should **pick the best match based on the arguments**, and *then* disallow static calls that are through an instance\! The actual result here is therefore (3):  

error CS0176: Member 'C.M(string)' cannot be accessed with an instance reference; qualify it with a type name instead

What is up with this craziness? If you believe that the rule “static methods should never be accessed through instances” is a good rule – and it seems reasonable – then why doesn’t overload resolution remove the static methods from the candidate set when called through an instance? Why does it even allow the “string” version to be an applicable candidate in the first place, if the compiler knows that it can never possibly work? I agree that this seems really goofy at first. To explain why this is not quite as dumb as it seems, consider the following variation on the problem. Class C stays the same.  

class B  
{  
  public C C = new C();  
  public void N()  
  {  
      C.M("hello");  
  }  
}

What does a call to N on an instance of B do?

(1) writes static M(string)  
(2) writes M(object)  
(3) compilation error  
(4) something else

.

.

.

.

.

.

A bit trickier now, isn’t it? Does C.M mean “call instance method M on the instance stored in this.C?” or does it mean “call static method M on type C”? Both are applicable\!

**Because of our goofy rule we do the right thing in this case.** We first resolve the call based solely on the arguments and determine that the static method is the best match. Then we check to see if the “receiver” at the call site can be interpreted as being through the type. It can. So we make the static call and write “static M(string)”. **If the instance version had been the best match then we would successfully call the instance method through the property.**

So the reason that the compiler does not remove static methods when calling through an instance is because the compiler does not necessarily know that you are calling through an instance. Because there are situations where it is ambiguous whether you’re calling through an instance or a type, we defer deciding which you meant until the best method has been selected.

Now, one could make the argument that in our first case, the receiver cannot *possibly* be a type. We *could* have further complicated the language semantics by having *three* overload resolution rules, one for when the receiver is *known to be a type*, one for when the receiver is *known to not be a type*, and one for when the receiver *might or might not be a type*. But rather than complicate the language further, we made the pragmatic choice of simply deferring the staticness check until after the bestness check. That’s not perfect but it is good enough for most cases and it solves the common problem.

The common problem is the situation that arises when you have a **property of a type with the same name**, and then try to call either a static method on the type, or an instance method on the property. (This can also happen with locals and fields, but local and field names typically differ in case from the names of their types.) The canonical motivating example is a public property named Color of type Color, so we call this “**the Color Color problem**”.

In real situations the instance and static methods typically have different names, so in the typical scenario you never end up calling an instance method when you expect to call a static method, or vice versa. The case I presented today with a name collision between a static and an instance method is relatively rare.

If you’re interested in reading the exact rules for how the compiler deals with the Color Color problem, see section 7.5.4.1 of the C\# 3.0 specification.

