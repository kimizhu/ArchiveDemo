# Even More Conversion Trivia

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/30/2007 5:10:00 PM

-----

I learn something new about C\# every day. This is a subtle and tricky language.

Pop quiz: foo is of a type which has a base class which has made a normal, everyday virtual override of System.Object.ToString. There are no additional overrides, hides, etc of ToString anywhere in foo's inheritance hierarchy. No ToString extension methods, etc. Under what circumstances do foo.ToString() and ((object)foo).ToString() produce different results? Remember, it is a virtual function, so the cast should not make a difference in what function is actually called.

As it turns out, my mistaken belief that the two of these have the same semantics caused a bug in the part of the compiler where we translate an expression tree lambda into a call to the Lambda\<T\> factory. That in turn was masking an even more subtle bug in the dynamic expression tree compiler.

Anyone out there have a guess as to what type and value of foo produces different behaviour here? I'll post the answer next time along with some discussion of the codegen issue.

