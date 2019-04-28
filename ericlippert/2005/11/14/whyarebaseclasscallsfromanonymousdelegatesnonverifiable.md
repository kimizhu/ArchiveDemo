# Why are base class calls from anonymous delegates nonverifiable?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/14/2005 2:28:00 PM

-----

I'm still learning my way around the C\# codebase – heck, I'm still learning my way around the Jscript codebase and I've been working on it for nine years, not nine weeks. Here's something I stumbled across while refactoring the anonymous method binding code last week that I thought might be interesting to you folks. Consider this simple program: using System;  
public delegate void D ();  
public class Alpha {  
  public virtual void Blah() {  
    Console.WriteLine("Alpha.Blah");  
  }  
}  
public class Bravo : Alpha {  
  public override void Blah() {  
    Console.WriteLine("Bravo.Blah");  
    base.Blah();  
  }  
} Pretty straightforward so far. Any **virtual** call to Blah on an instance of Bravo does a **non-virtual** call to the base class implementation. Now let's do a similar thing from an anonymous delegate in Bravo:   public void Charlie() {  
    int x = 123;  
    D d = delegate {  
      this.Blah();  
      base.Blah();  
      Console.WriteLine(x);  
    };  
    d();  
  } When you compile this thing up you get a crazy-sounding warning: warning CS1911: Access to member 'Alpha.Blah()' through a 'base' keyword from an anonymous method or iterator results in unverifiable code. Consider moving the access into a helper method on the containing type. And indeed, if you compile this up and run it through PEVERIFY.EXE, sure enough you'll get an unverifiable code warning. Unverifiable code requires full trust and is generally to be avoided – what is going on here? This is an artefact of the way that C\# realizes anonymous delegates. The anonymous delegate above captures both this and x, and therefore we actually generate code that would look something like this if you decompiled it: public class Bravo : Alpha {  
  public override void Blah() {  
    Console.WriteLine("Bravo.Blah");  
    base.Blah();  
  }  
  private class \_\_locals {  
    public int \_\_x;  
    public Bravo \_\_this;  
    public void \_\_method() {  
      this.\_\_this.blah();  
      \_\_nonvirtual\_\_ ((Alpha)this.\_\_this).Blah());  
      Console.WriteLine(this.\_\_x);  
    }  
  }  
  public void Charlie() {  
    \_\_locals locals = new \_\_locals();  
    locals.\_\_x = 123;  
    locals.\_\_this = this;  
    D d = new D(locals.\_\_method);  
    d();  
  }  
} Of course there is no such thing in C\# as \_\_nonvirtual\_\_, so really there is no way to decompile this thing back into real C\#.  Which is exactly the problem\! What's happening here is that we are doing a non-virtual call on a this reference which is not the "real" this reference of the call. That's a pretty suspicious programming practice.  People who build object hierarchies that manipulate sensitive information have a reasonable expectation that the *only* way to call a base class implementation of a virtual method is from the derived class itself, not from some third-party method.  Therefore the CLR treats this as unverifiable code, and therefore we have to issue a warning. (I suppose the CLR team could have made an exception for nonvirtual references from classes scoped to within the class in question, or for that matter, classes from the same assembly, but they didn't.) Now, I was refactoring the code that generates the captured variable class, and so of course I wanted to write a few additional unit tests to make sure that I wasn't breaking anything. When I ran into this warning the first unit test that I wrote was actually a somewhat simplified version of the above:   public void Charlie() {  
    D d = delegate {  
      this.Blah();  
      base.Blah();  
    };  
    d();  
  } This issues the warning above, but when I ran the generated executable through PEVERIFY I was surprised to discover that it verified just fine. As it turns out, the compiler is clever about this one. It sees that the only captured variable is the this reference, and therefore it can optimize away the locals class.  For this case it simply generates the anonymous method as just another method on the Bravo class. Since this is a method of Bravo, not a special locals class, the nonvirtual call is on the real this reference, so it is verifiable. We decided to issue the warning even in this case because we thought that it would make C\# seem really weird and brittle to suddenly start issuing the warning when you add an additional outer local variable reference to the anonymous delegate. **Even when this doesn't actually generate nonverifiable code, it's a good idea to get in the habit of creating a helper method on the real class that does the nonvirtual access.** Of course, all of the stuff above is implementation details. **You cannot rely upon future versions of C\# to continue to generate anonymous methods in this manner.** We probably will, but who knows what new features will be added to the CLR that might make it possible to not generate a bunch of hidden classes behind the scenes to do this work? Please do not attempt to do anything silly like reflecting upon the class to discover the hidden nested classes and use them; you're just asking for future pain if you do.

