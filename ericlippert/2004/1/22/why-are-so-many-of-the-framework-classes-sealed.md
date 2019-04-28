<div id="page">

# Why Are So Many Of The Framework Classes Sealed?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/22/2004 12:45:00 PM

-----

<div id="content">

<span><span><span><span> </span></span></span></span>

<span>I talked [earlier this month](http://blogs.msdn.com/ericlippert/archive/2004/01/07/48399.aspx) about some issues in subclassing, and recommended sealing your classes. A Joel On Software reader asks why Microsoft ships so many sealed classes in the framework.<span>  </span>The poster said: "[I've yet to see a reasonable explanation about why they limited flexibility in such a way.](http://discuss.fogcreek.com/joelonsoftware/default.asp?cmd=show&ixPost=106245&ixReplies=9) " </span>

<span></span>

<span>Well, every public class that my team produces is sealed if possible.<span>  </span>If it is not possible to seal a class then, if possible, it has an inheritance demand on it so that only someone with the MSFT private key can subclass it.<span>  </span>My reasons for insisting upon this policy boil down to one overriding principle: </span>

<span></span>

**<span>Good code does exactly what it was designed to do, no more, no less. </span>**

<span></span>

<span>Let me expand upon that in four ways. </span>

<span></span>

<span>1) **Philosophical**.<span>  </span>OOP design includes subclassing to represent the polymorphic "is a" relationship between two things.<span>  </span>A Giraffe IS AN Ungulate IS A Mammal IS AN Animal...<span>  </span>Unless I can think of a clear case where a customer would need to express an IS A relationship with some code that I produce, I don't allow for such cases. </span>

<span></span>

<span>2) **Practical**.<span>  </span>Designing classes so that they can be effectively extended by third parties is HARD.<span>  </span>(Look at the collection base classes for example.) You have to get the design right -- what is protected?<span>  </span>You have to implement that design correctly. The test matrix grows enormously because you have to think about what weird things people are going to do. You have to document the protected methods and write documentation on how to properly subclass the thing.<span>  </span> </span>

<span></span>

<span>This is all expensive and time consuming -- that is time that we could be spending looking for bugs in more important user scenarios, planning future versions, fixing security holes, whatever.<span>  </span>[There is only a finite amount of developer time we can spend on designing and implementing code, so we have to spend it the way that benefits customers most.](http://blogs.msdn.com/ericlippert/archive/2003/10/28/53298.aspx)<span>  </span>If the class is not designed to be extended, I'm going to avoid all that expense by sealing it.<span>  </span>I am not going to release half-baked classes that look extensible but in fact are not quite there. </span>

<span></span>

<span>3) **Compatible**.<span>  </span>If in the future I discover that I should have sealed a class, I'm stuck.<span>  </span>Sealing a class is a breaking change.<span>  </span>If I discover that I should have left a class unsealed, unsealing in a future version is a non-breaking change.<span>  </span>Sealing classes helps maintain compatibility. </span>

<span></span>

<span>4) **Secure**. the whole point of polymorphism is that you can pass around objects that look like Animals but are in fact Giraffes.<span>  </span>There are potential security issues here. </span>

<span></span>

<span>Every time you implement a method which takes an instance of an unsealed type, you MUST write that method to be robust in the face of potentially hostile instances of that type.<span>  </span>You cannot rely upon any invariants which you know to be true of YOUR implementations, because some hostile web page might subclass your implementation, override the virtual methods to do stuff that messes up your logic, and passes it in.<span>  </span>Every time I seal a class, I can write methods that use that class with the confidence that I know what that class does.</span><span> </span>

-----

<span>Now, I recognize that developers are highly practical people who just want to get stuff done.<span>  </span>Being able to extend any class is convenient, sure.<span>  </span>Typical developers say "IS-A-SHMIZ-A, I just want to slap a Confusticator into the Froboznicator class".<span>  </span>That developer could write up a hash table to map one to the other, but then you have to worry about when to remove the items, etc, etc, etc -- it's not rocket science, but it is work. </span>

<span></span>

<span>Obviously there is a tradeoff here.<span>  </span>The tradeoff is between letting developers save a little time by allowing them to treat any old object as a property bag on the one hand, and developing a well-designed, OOPtacular, fully-featured, robust, secure, predictable, testable framework in a reasonable amount of time -- and I'm going to lean heavily towards the latter.<span>  </span>Because you know what?<span>  </span>Those same developers are going to complain bitterly if the framework we give them slows them down because it is half-baked, brittle, insecure, and not fully tested\! </span><span><span>  </span></span>

<span></span>

</div>

</div>

