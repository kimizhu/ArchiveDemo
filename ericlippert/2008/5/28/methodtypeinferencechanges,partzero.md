# Method Type Inference Changes, Part Zero

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/28/2008 11:16:00 AM

-----

Back in November I wrote a bit about a [corner case in method type inference](http://blogs.msdn.com/ericlippert/archive/2007/11/05/c-3-0-return-type-inference-does-not-work-on-member-groups.aspx) which does not work as expected or as specified in C\# 3.0. A number of people made blog comments, sent me mail, and entered "Connect" issues with additional problems and ideas for how we could improve this algorithm. (Particular thanks to nikov for his many fascinating and detailed Connect reports.) As a result, as soon as I had time I embarked upon a detailed review of the method type inference specification and the implementation.

The good news is that this resulted in a complete overhaul of both the implementation and the specification; they are now consistent with each other and to the best of my knowledge, correct. Furthermore, the principle concern of commenters on my earlier article has been dealt with: return type inference from a method group to the return type of a delegate type will be legal, but only when the delegate type's input parameters are all completely known. This eliminates the "chicken and egg" problem I discussed in November, whereby overload resolution on the method group must consume the very same unfixed formal parameter types that it is attempting to infer.

The bad news is that these changes to the implementation will not make it in to the service release of C\# 3.0; they will have to wait for a later revision of the compiler. (It is not yet clear to me whether or not the changes to the specification will make it into the next edition of the published specification.)

This is one of the most complex areas of the specification and implementation; I'd like to spend some time in this blog going over the specification in detail and explaining where we got things subtly wrong, and how we intend to fix it.

In this series I want to hit on the following points:

  - What did method type inference look like in C\# 2.0? Why was it inadequate for LINQ?
  - How did we attempt to modify and ultimately rewrite the specification for C\# 3.0?
  - Where did we go subtly wrong in the specification and the implementation?

Next time, we'll get started with a look back at C\# 2.0.

