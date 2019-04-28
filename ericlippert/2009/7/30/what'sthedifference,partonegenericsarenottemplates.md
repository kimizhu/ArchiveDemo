# What's the difference, part one: Generics are not templates

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/30/2009 9:58:00 AM

-----

Because I'm a geek, I enjoy learning about the sometimes-subtle differences between easily-confused things. For example:

  - I'm still not super-clear in my head on the differences between a **hub**, **router** and **switch** and how it relates to the gnomes that live inside of each.
  - Hunks of minerals found in nature are **rocks**; as soon as you put them in a garden or build a bridge out of them, suddenly they become **stones**.
  - When a **pig** hits 120 pounds, it's a **hog**.

I thought I might do an occasional series on easily confounded concepts in programming language design. 

Here’s a question I get fairly often:

 

public class C  
{  
  public static void DoIt\<T\>(T t)  
  {  
    ReallyDoIt(t);  
  }  
  private static void ReallyDoIt(string s)  
  {  
    System.Console.WriteLine("string");  
  }  
  private static void ReallyDoIt\<T\>(T t)  
  {  
    System.Console.WriteLine("everything else");  
  }  
}

What happens when you call C.DoIt\<string\>? Many people expected that “string” is printed, when in fact “everything else” is always printed, no matter what T is.

The C\# specification says that when you have a choice between calling ReallyDoIt\<string\>(string) and ReallyDoIt(string) – that is, when the choice is between two methods that have identical signatures, but one gets that signature via generic substitution – then we pick the “natural” signature over the “substituted” signature. Why don’t we do that in this case?

Because that’s not the choice that is presented. If you had said

 

ReallyDoIt("hello world");

then we would pick the “natural” version. But you didn’t pass something known to the compiler to be a string. You passed something known to be a T, an unconstrained type parameter, and hence it could be anything. So, the overload resolution algorithm reasons, is there a method that can always take anything? Yes, there is.

This illustrates that **generics** in C\# are not like **templates** in C++. You can think of templates as a fancy-pants search-and-replace mechanism. When you say DoIt\<string\> in a template, the compiler conceptually searches out all uses of “T”, replaces them with “string”, and then compiles the resulting source code. Overload resolution proceeds with the substituted type arguments known, and the generated code then reflects the results of that overload resolution.

That’s not how generic types work; generic types are, well, *generic*. We do the overload resolution **once** and bake in the result. We do not change it at runtime when someone, possibly in an entirely different assembly, uses string as a type argument to the method. The IL we’ve generated for the generic type already has the method its going to call picked out. The jitter does not say “well, I happen to know that if we asked the C\# compiler to execute right now with this additional information then it would have picked a different overload. Let me rewrite the generated code to ignore the code that the C\# compiler originally generated...” The jitter knows nothing about the rules of C\#.

Essentially, the case above is no different from this:

 

public class C  
{  
  public static void DoIt(object t)  
  {  
    ReallyDoIt(t);  
  }  
  private static void ReallyDoIt(string s)  
  {  
    System.Console.WriteLine("string");  
  }  
  private static void ReallyDoIt(object t)  
  {  
    System.Console.WriteLine("everything else");  
  }  
}

When the compiler generates the code for the call to ReallyDoIt, it picks the object version because that’s the best it can do. If someone calls this with a string, then it still goes to the object version.

Now, if you do want overload resolution to be re-executed at runtime based on the runtime types of the arguments, we can do that for you; that’s what the new “dynamic” feature does in C\# 4.0. Just replace “object” with “dynamic” and when you make a call involving that object, we’ll run the overload resolution algorithm at runtime and dynamically spit code that calls the method that the compiler would have picked, had it known all the runtime types at compile time.

