# An Inheritance Puzzle, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/30/2007 7:00:00 AM

-----

Today, the answer to [Friday's puzzle](http://blogs.msdn.com/ericlippert/archive/2007/07/27/an-inheritance-puzzle-part-one.aspx). It prints "Int32". But why?

Some readers hypothesized that M would print out "Int32" because the declaration B : A\<int\> somehow tells B that T is to be treated as int, now and forever. Though the *answer* is right, the *explanation* is not quite right. One can illustrate this by taking C out of the picture. If you say (new A\<string\>.B()).M(), you'll get "String". The fact that B is an A\<int\> doesn't make T always int inside B\!

The always-keen Stuart Ballard was the first to put his finger upon the real crux of the problem -- but he's seen this problem before, so he had an advantage. Eamon Nerbonne was the first to post a complete and correct explanation.

The really thorny issue here is that the declaration class C : B is upon close inspection, somewhat ambiguous. Is that equivalent to class C : A\<T\>.B or class C : A\<int\>.B?

Clearly it matters which we choose. A method group can only be treated as a member if the method is on a *base class*. Merely being on an *outer class* doesn't cut it:

 

public class X { public void M(){} }  
public class Y {  
  public void N() {}  
  public class Z : X {}  
}  
...  
(new Y.Z()).M(); // legal, from base class  
(new Y.Z()).N(); // illegal -- outer class members are not members of inner classes.

In our example, when we called M on an instance of A\<string\>.B.C, it was calling a method of the base class of C. If the base class of C is A\<T\>.B, then that should call A\<T\>.B.M, and it should print out whatever the current value of T is -- in this case, "String". If the base class of C is A\<int\>.B, then that should call A\<int\>.B.M, so it should print out "Int32".

We choose the latter as the base class. That was certainly a surprise to me. And Stuart Ballard. And, amusingly enough, when I sprang this one upon Anders and didn't give him time to think about it carefully, it was a surprise to him as well. When I sprang it on Cyrus, he cheerfully pointed out that he already posted [a harder version of this problem back in 2005](http://blogs.msdn.com/cyrusn/archive/2005/08/01/446431.aspx), the solution of which Stuart characterized back then as "Insanely complex, but it makes perfect sense." I couldn't agree more, though at least my version of the puzzle is somewhat simpler.

Anyway, why on earth ought that to be the case? Surely the B in class C : B means the immediately containing class, which is A\<T\>.B, not A\<int\>.B\! And yet it does not.

These generics are screwing up our intuitions. Let's look at an example which has no generics at all:

 

public class D {  
    public class E {}  
}  
public class F {  
    public class E { }  
    public class G {  
        public E e; // clearly F.E  
    }  
}  
public class H : D {  
    public E e; // clearly D.E  
}

This is all legal so far, and should be pretty clear. When we are binding a name to a type, the type we get is allowed to be *a member of any base class* ***or*** *a member of any outer class*. But what if we have *both* to choose from?

 

public class J {  
    public class E {}  
    public class K : D {  
        public E e; // Is this J.E or D.E?  
    }  
}

We could just throw up our hands and say that this is ambiguous and therefore illegal, but we'd rather not do that if we can avoid it. We have to prefer one of them, and **we've decided that we will give priority to base classes over outer classes**. Derived classes have an "is a kind of" relationship with their base classes, and that is logically a "tighter" binding than the "is contained in" relationship that inner classes have with outer classes.

Another way to think about it is that all the members you get from your base class are all "in the current scope"; therefore all the members you get from outer scopes are given lower priority, since *stuff inside inner scopes takes priority over stuff in outer scopes*.

The algorithm we use to search for a name used in the context of a type S is as follows:

  - search S's type parameters

  - search S's accessible inner classes

  - search accessible inner classes of all of S's base classes, going in order from most to least derived

  - S←S's outer class, start over

(And if that fails then we invoke the whole mechanism of searching the namespaces that are in scope, checking alias clauses, etc.)

With that in mind, now the solution should finally make some sense. At the point where we are resolving the base class of C we know that C has no type parameters. We do not know what the base class or inner classes of C are -- that's what we're trying to figure out -- so we skip checking them.

The next thing we check is the outer class, which is A\<T\>.B, but we do NOT say, aha, the outer class is called B, we're done. That is not at all what the algorithm above says. Instead, it says check A\<T\>.B to see if it has a type variable called B or an inner type called B. It does not, so we keep searching.

The base type of A\<T\>.B is A\<int\>. The outer type of A\<T\>.B is A\<T\>. Both have an accessible inner class called B. Which do we pick? **The base type gets searched first**, so B resolves to A\<int\>.B. Obviously.

Having members of base classes bind tighter than members from outer scopes can lead to bizarre situations but they are generally pretty contrived. For example:

 

public class K { public class L {} }  
public class L : K {  
  L myL; // this is K.L\!  
}

And of course, you can always get around these problems by eliminating the ambiguity:

 

public class A\<T\> {  
  public class B : A\<int\> {  
    public void M() { ... }  
    public class C : A\<T\>.B { } // no longer ambiguous which B is picked.

Finally, the specification of this behaviour is a bit tricky to understand:

 

Otherwise, if T contains a nested accessible type having name I and K type parameters, then the namespace-or-type-name refers to that type constructed with the given type arguments. If there is more than one such type, the type declared within the more derived type is selected.

By "if T contains a nested accessible type", it means "if T, *or any of its base classes*, contains a nested accessible type". I completely failed to comprehend that the first n times I read that section. I'll see if I can get that clarified in the next version of the standard.

