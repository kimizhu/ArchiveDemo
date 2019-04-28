# Why Can't I Access A Protected Member From A Derived Class?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/9/2005 5:20:00 PM

-----

A question I got recently was about access to protected methods from a derived class. Clearly that's what "protected" means – that you can access it from a derived class. In that case, why doesn't this work? class Ungulate {  
  protected void Eat() { /\* whatever \*/ }  
} class Giraffe : Ungulate {  
  public static void FeedThem() {  
    Giraffe g1 = new Giraffe();  
    Ungulate g2 = new Giraffe();  
    g1.Eat(); // fine  
    g2.Eat(); // compile-time error "Cannot access protected member"  
  }  
} What the heck? Giraffe is derived from Ungulate, so why can't it always call the protected method? To understand, you have to think like the compiler. The compiler can only reason from the *static* type information, not from the fact that we know that at runtime g2 actually will be a Giraffe. For all the compiler knows from the static type analysis, what we've actually got here is class Ungulate {  
  protected virtual void Eat() { /\* whatever \*/ }  
} class Zebra : Ungulate {  
  protected override void Eat() { /\* whatever \*/ }  
} class Giraffe : Ungulate {  
  public static void FeedThem() {  
    Giraffe g1 = new Giraffe();  
    Ungulate g2 = new Zebra();  
    g1.Eat(); // fine  
    g2.Eat(); // compile-time error "Cannot access protected member"  
  }  
} We can call Ungulate.Eat legally from Giraffe, but we can't call the protected method Zebra.Eat from anything except Zebra or a subclass of Zebra. Since the compiler cannot determine from the static analysis that we are *not* in this illegal situation, it must flag it as being illegal. Incidentally, C++ has the same rule.

