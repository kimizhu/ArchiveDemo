# Hide and seek

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/14/2010 8:40:00 AM

-----

Another [interesting question](http://stackoverflow.com/questions/2845276/should-a-protected-property-in-a-c-child-class-hide-access-to-a-public-property) from StackOverflow. That thing is a gold mine for blog topics. Consider the following:

class B  
{  
  public int X() { return 123; }  
}  
class D : B  
{  
  new protected int X() { return 456; }  
}  
class E : D  
{  
  public int Y() { return X(); } // returns 456  
}  
class P  
{  
  public static void Main()  
  {  
    D d = new D();  
    Console.WriteLine(d.X());  
  }  
}

There are two possible behaviours here. We could resolve X to be B.X and compile successfully, or resolve X to be D.X and give a "you can't access a protected method of D from inside class Program" error.

\[UPDATE: I've clarified this portion of the text to address questions from the comments. Thanks for the good questions.\]

We do the former.To compute the set of possible resolutions of name lookup,  the spec says"*the set consists of all accessible members named N in T, including inherited members*" but D.X is not accessible from outside of D; it's protected. So D.X is not in the accessible set.

The spec then says "*members that are hidden by other members are removed from the set*". Is B.X hidden by anything? It certainly appears to be hidden by D.X. Well, let's check. The spec says "*A declaration of a new member hides an inherited member only within the scope of the new member.*" The declaration of D.X is only hiding B.X within its scope: the body of D and the bodies of declarations of types derived from D. Since P is neither of those, D.X is not hiding B.X there, so B.X is visible, so that's the one we choose.

Inside E, D.X is accessible and hides B.X, so D.X is in the set and B.X is not.

What's the justification for this choice? Doesn't this conflict with our rule that methods in more derived classes are better than methods in base classes?

No, it doesn't. Remember, the rule about prefering derived to base methods is to mitigate the brittle base class problem. So is this rule\! Consider this brittle base class scenario:

FooCorp makes Foo.DLL:  
  
public class Foo  
{  
  public object Blah() { ... }  
} 

BarCorp makes Bar.DLL:

public class Bar : Foo  
{  
  // stuff not having to do with Blah  
} 

 ABCCorp makes ABC.EXE:

 public class ABC  
{  
  static void Main()  
  {  
    Console.WriteLine((new Bar()).Blah());   
  }  
}  

Now BarCorp says "You know, in our internal code we can guarantee that Blah only ever returns string thanks to our knowledge of our derived implementation. Let's take advantage of that fact in our internal code."

public class Bar : Foo  
{  
  internal new string Blah()  
  {  
    object r = base.Blah();  
    Debug.Assert(r is string);  
    return (string)r;  
  }  
} 

ABCCorp picks up a new version of Bar.DLL which has a bunch of bug fixes that are blocking them. **Should their build break because they have a call to Blah, an internal method on Bar?** Of course not. That would be terrible. This change is a *private implementation detail that should be invisible outside of Bar.DLL*. The fact that hiding methods are ignored outside of their scopes means that they can be safely used for internal implementation details without breaking downstream users.

 

(Eric is in Oslo at NDC; this posting was prerecorded.)

