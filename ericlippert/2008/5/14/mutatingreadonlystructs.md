# Mutating Readonly Structs

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/14/2008 10:10:00 AM

-----

Consider this program which attempts to mutate a readonly mutable struct. What happens?

 

struct Mutable {  
    private int x;  
    public int Mutate() {  
        this.x = this.x + 1;  
        return this.x;  
    }  
}

class Test {  
    public readonly Mutable m = new Mutable();  
    static void Main(string\[\] args) {  
        Test t = new Test();  
        System.Console.WriteLine(t.m.Mutate());  
        System.Console.WriteLine(t.m.Mutate());  
        System.Console.WriteLine(t.m.Mutate());  
    }  
}

There are a number of things this program could do. Does it:

1) Print 1, 2, 3 -- because m is readonly, but the "readonly" only applies to m, not to its contents.  
2\) Print 0, 0, 0 -- because m is readonly, x cannot be changed. It always has its default value of zero.  
3\) Throw an exception at runtime, when the attempt is made to mutate the contents of a readonly field.  
4\) Do something else

?

People are frequently surprised to learn that the answer is (4). In fact, this prints 1, 1, 1. 

Why?

Because, remember, accessing a value type gives you a COPY of the value. When you say t.m, you get a copy of whatever is presently stored in m. m is immutable, but the copy is not. The copy is then mutated, and the value of x in the copy is returned. But m remains untouched.

The relevant section of the specification is 7.5.4, which states that when resolving "E.I" where E is an object and I is a field...

 

...if the field is readonly and the reference occurs outside an instance constructor of the class in which the field is declared, then the result is a value, namely the value of the field I in the object referenced by E.

The important word here is that the result is the *value* of the field, not the *variable* associated with the field. *Readonly fields are not variables outside of the constructor*. ([The initializer here is considered to be inside the constructor; see my earlier post on that subject](http://blogs.msdn.com/ericlippert/archive/2008/02/15/why-do-initializers-run-in-the-opposite-order-as-constructors-part-one.aspx).)

Great. What about that second dot, as in ".Mutate()"?  We look at section 7.4.4 to find out how to invoke E.M():

 

If E is not classified as a variable, then a temporary local variable of E's type is created and the value of E is assigned to that variable. E is then reclassified as a reference to that temporary local variable. The temporary variable is accessible as *this* within M, but not in any other way. Thus, only when E is a true variable is it possible for the caller to observe the changes that M makes to *this*.

And there you go. Value semantics are tricky\!

This is yet another reason why mutable value types are evil. Try to always make value types immutable.

