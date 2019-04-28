# JScript, IE Behaviours and instanceof Redux

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/4/2004 10:50:00 AM

-----

 

Today, a quick follow-up on my [earlier post](/ericlippert/archive/2003/11/06/53352.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/06/53352.aspx") about the instanceof operator in JScript. A customer asked me recently about the following scenario:  a script on a web page calls a script function in a behaviour, which returns an object of type Array.  But when she uses the instanceof operator, it reports that the object is not an Array.  What's going on? 

Well, as it turns out, in IE6** the browser creates two entirely separate script engines, one for the HTC, one for the main page**.  That means that each engine has its own independent copy of the built-in script objects, and the Array prototype is one of them. 

In case it's not clear, let me draw that out for you in more detail.  Let's give every object a name based on what engine created it, so that we can tell them apart. 

Engine Alpha, for the "main" .HTM file contains objects ArrayConstructorAlpha.  ArrayConstructorAlpha has prototype property ArrayPrototypeAlpha. Therefore, any instance of Array created in engine Alpha will have prototype ArrayPrototypeAlpha. 

Similarly, Engine Beta, for the "behaviour" .HTC file contains ArrayConstructorBeta.  ArrayConstructorBeta has prototype property ArrayObjectBeta. Therefore, any instance of Array created by engine Beta will have prototype ArrayPrototypeBeta.** **

Consider then what happens when you pass an instance of Array from Beta to Alpha, and run code in Alpha that looks like "betaArray instanceof Array".  In engine Alpha, Array will be resolved as ArrayConstructorAlpha.  Therefore you're actually asking "does the object have anywhere on its prototype chain the prototype property of ArrayConstructorAlpha, which is ArrayPrototypeAlpha?" 

Clearly it does not. 

I'm sure that you will agree with me that this is the only sensible possible behaviour given the semantics of the instanceof operator.  Suppose for example the HTC had this line of code: 

Array.prototype.blah = 123; 

and the HTM had this line: 

Array.prototype.blah = 456; 

Now clearly two instances of Array, one from each engine, are not both instances of the same constructor; they each have a prototype with a **different** blah property.  Thus at most one of them can be an instance of one of the Array constructors. 

That then leaves us with the question of why there are two engines in the first place.  Why doesn't the behaviour simply inject its code into the containing page's script engine? 

Well, there you've got me.  I don't know why IE made this particular design decision.  I could guess -- like, perhaps this is to resolve the problem with the scenario where the behaviour and the main page both have a function with the same name.  Hmm -- but that could also be resolved by putting the behaviour code into a module.  (Which reminds me, I should do a post about module semantics in the script engines.)  

The long and the short of it is that once again we've shown that **it is hard to determine the type of a JScript object**.  JScript has a very fluid notion of what a type is\!  Objects are extremely dynamic bags of properties in JScript, so its not even that helpful to know what type an object is, because types give you no guarantee of any particular behaviour.   In JScript .NET in fast mode, where prototypes are immutable, the type system becomes a much more useful tool, as types actually do constrain behaviour.

