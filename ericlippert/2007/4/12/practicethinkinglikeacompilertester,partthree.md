# Practice thinking like a compiler tester, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/12/2007 10:00:00 AM

-----

[Yesterday I posed a slightly harder version](http://blogs.msdn.com/ericlippert/archive/2007/04/11/practice-thinking-like-a-compiler-tester-part-two.aspx) of the [puzzle I posted the day before](http://blogs.msdn.com/ericlippert/archive/2007/04/10/practice-thinking-like-a-compiler-tester.aspx). 

Reader Steve found a solution: 

 

public class C : A {}  
public class A {  
    public class D : C {}  
}

See his [comment](http://blogs.msdn.com/ericlippert/archive/2007/04/11/practice-thinking-like-a-compiler-tester-part-two.aspx#2087926) for the trace of the logic that shows why this asserts.  

The scenario that our testers found which triggered the assertion is just slightly more complex, but basically the same idea:

 

public class C : A.B {}  
public class A {  
    public class B {}  
    public class D : C {}  
}

Tricky\! 

Fortunately, the problem was that the assertion and comment was wrong, not the program logic.

