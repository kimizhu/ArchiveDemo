<div id="page">

# JScript, IE Behaviours and instanceof Redux

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/4/2004 10:50:00 AM

-----

<div id="content">

<span> </span>

<span>Today, a quick follow-up on my [earlier post](/ericlippert/archive/2003/11/06/53352.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/06/53352.aspx") about the </span><span>instanceof</span><span> operator in JScript. A customer asked me recently about the following scenario:  a script on a web page calls a script function in a behaviour, which returns an object of type </span><span>Array</span><span>.  But when she uses the </span><span>instanceof</span><span> operator, it reports that the object is not an </span><span>Array</span><span>.  What's going on? </span>

<span></span>

<span>Well, as it turns out, in IE6**<span> the browser creates two entirely separate script engines, one for the HTC, one for the main page</span>**.  That means that each engine has its own independent copy of the built-in script objects, and the </span><span>Array</span><span> prototype is one of them. </span>

<span></span>

<span>In case it's not clear, let me draw that out for you in more detail.  Let's give every object a name based on what engine created it, so that we can tell them apart. </span>

<span></span>

<span>Engine Alpha, for the "main" .HTM file contains objects </span><span>ArrayConstructorAlpha</span><span>.  </span><span>ArrayConstructorAlpha</span><span> has </span><span>prototype</span><span> property </span><span>ArrayPrototypeAlpha</span><span>. Therefore, any instance of </span><span>Array</span><span> created in engine </span><span>Alpha</span><span> will have prototype </span><span>ArrayPrototypeAlpha</span><span>. </span>

<span></span>

<span>Similarly, Engine Beta, for the "behaviour" .HTC file contains </span><span>ArrayConstructorBeta</span><span>.  </span><span>ArrayConstructorBeta</span><span> has </span><span>prototype</span><span> property </span><span>ArrayObjectBeta</span><span>. Therefore, any instance of </span><span>Array</span><span> created by engine Beta will have prototype </span><span>ArrayPrototypeBeta</span><span>.**<span> </span>**</span>

<span></span>

<span>Consider then what happens when you pass an instance of Array from Beta to Alpha, and run code in Alpha that looks like "</span><span>betaArray instanceof Array</span><span>".  In engine Alpha, </span><span>Array</span><span> will be resolved as </span><span>ArrayConstructorAlpha</span><span>.  Therefore you're actually asking "does the object have anywhere on its prototype chain the </span><span>prototype</span><span> property of </span><span>ArrayConstructorAlpha</span><span>, which is </span><span>ArrayPrototypeAlpha</span><span>?" </span>

<span>Clearly it does not. </span>

<span></span>

<span>I'm sure that you will agree with me that this is the only sensible possible behaviour given the semantics of the </span><span>instanceof</span><span> operator.  Suppose for example the HTC had this line of code: </span>

<span></span>

<span>Array.prototype.blah = 123; </span>

<span></span>

<span>and the HTM had this line: </span>

<span></span>

<span>Array.prototype.blah = 456; </span>

<span></span>

<span>Now clearly two instances of </span><span>Array</span><span>, one from each engine, are not both instances of the same constructor; they each have a prototype with a **<span>different</span>** blah property.  Thus at most one of them can be an instance of one of the </span><span>Array</span><span> constructors. </span>

<span></span>

<span>That then leaves us with the question of why there are two engines in the first place.  Why doesn't the behaviour simply inject its code into the containing page's script engine? </span>

<span></span>

<span>Well, there you've got me.  I don't know why IE made this particular design decision.  I could guess -- like, perhaps this is to resolve the problem with the scenario where the behaviour and the main page both have a function with the same name.  Hmm -- but that could also be resolved by putting the behaviour code into a module.  (Which reminds me, I should do a post about module semantics in the script engines.)  </span>

<span></span><span>The long and the short of it is that once again we've shown that **it is hard to determine the type of a JScript object**.  JScript has a very fluid notion of what a type is\!  Objects are extremely dynamic bags of properties in JScript, so its not even that helpful to know what type an object is, because types give you no guarantee of any particular behaviour.   In JScript .NET in fast mode, where prototypes are immutable, the type system becomes a much more useful tool, as types actually do constrain behaviour.</span>

</div>

</div>

