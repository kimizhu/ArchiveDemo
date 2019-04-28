# Future Breaking Changes, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/4/2007 10:30:00 AM

-----

As I said [earlier](http://blogs.msdn.com/ericlippert/archive/2007/08/31/future-breaking-changes-part-two.aspx), we hate causing breaking changes in our product, the C\# compiler, because they cause our customers pain.

Said customers are also software developers, and presumably they hate causing breaking changes for their customers as much as we do. We want to throw our customers into the [Pit of Success](http://www.codinghorror.com/blog/archives/000940.html) and give them tools which encourage them where possible to prevent breaking changes. This leads to some subtle issues in language design.

Pop quiz. What does this program do?

-----

// Alpha.DLL  
namespace Alpha {  
  public class Charlie {  
    public void Frob(int i) { System.Console.WriteLine("int"); }  
    // etc.  
  }  
} 

-----

// Bravo.EXE, references Alpha.DLL.  
namespace Bravo {  
  public class Delta : Alpha.Charlie {  
    public void Frob(float f) { System.Console.WriteLine("float"); }  
    // etc.  
    public static void Main() {  
      Delta d = new Delta();  
      d.Frob(1);  
    }  
  }  
}

-----

Most people look at this program and say “clearly Charlie.Frob(int) is the best possible match for the call, so that is called.” A compelling argument, but wrong. As the standard says, “methods in a base class are not candidates if any method in a derived class is applicable".

In other words, the overload resolution algorithm starts by searching the class for an applicable method. If it finds one then all the other applicable methods in deeper base classes are removed from the candidate set for overload resolution. Since Delta.Frob(float) is applicable, Charlie.Frob(int) is never even considered as a candidate. Only if no applicable candidates are found in the most derived type do we start looking at its base class.

Why on earth would we do that? Clearly in this example the base class member is the far better match, so why wouldn’t we even consider it?

It is instructive to consider what happens in a world where we do implement the rule “pick the best applicable candidate from any base”. Suppose we did that.

In the previous version of Alpha.DLL, Charlie did not have a method Frob(int).; When Bravo Corporation wrote Bravo.EXE, every call inside class Delta to method Frob was a call to Delta.Frob(float). Then one day Alpha corporation did customer research and discovered that a lot of their customers like to frob integers. They added this feature in their latest version. Delta corporation gets the new version of Alpha.DLL, recompiles Bravo.EXE, and suddenly their carefully developed code is sometimes calling a method that they didn’t write, which does something subtly incompatible with their implementation.

Alpha corporation has just pushed a breaking change onto Bravo corporation, which, if they don’t catch it in time, may now be pushing a subtly broken version onto their customers in turn, and hey\! we’re in the Pit of Despair again\!

This particular family of breaking changes is called the "brittle base class problem"; there are many versions of it and [different languages deal with it in different ways.](http://blogs.msdn.com/ericlippert/archive/2004/01/07/48399.aspx) Lots of work went into the design of C\# to try and make it harder for people to accidentally cause brittle base class problems. That is why we make you distinguish between the original definition of a virtual method and an overriding method. That is why we make you put “new” on methods which shadow other methods. All these semantics are in part to help prevent, mitigate or diagnose brittle base class issues and thereby prevent accidental breaking changes in C\# code.

Next time on FAIC: some psychic debugging. Then a bit later I want to talk more about breaking changes, this time in the context of thinking about covariance and contravariance.

