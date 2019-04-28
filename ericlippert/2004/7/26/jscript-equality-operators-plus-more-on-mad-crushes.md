<div id="page">

# JScript Equality Operators, plus More On Mad Crushes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/26/2004 12:01:00 PM

-----

<div id="content">

<span>First, the technical stuff.  Here's a recent question I received about missing data in JScript.  </span> <span></span>

> <span>I’ve seen it stated that in order to determine if a variable is undefined, you have to use </span><span>if("undefined" == typeof(x))  
> </span><span>I attended a presentation on JScript today where the presenter used </span><span>if(undefined == x)  
> </span><span>When I asked about that, he stated that it works and he is not aware of any problems with it. Is he correct?</span>

<span>I discussed related issues earlier ([here](http://weblogs.asp.net/ericlippert/archive/2003/09/30/53120.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/30/53120.aspx") and [here](http://weblogs.asp.net/ericlippert/archive/2003/10/01/53128.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/01/53128.aspx") and [here](http://weblogs.asp.net/ericlippert/archive/2003/11/05/53336.aspx "http://weblogs.asp.net/ericlippert/archive/2003/11/05/53336.aspx")), but I figure that this is another opportunity for clarification. </span>

<span></span> <span>There are three statements made.  </span>

<span>1) In order to determine if a variable is undefined, you **<span>have to</span>** use </span><span>if("undefined" == typeof(x))  
</span><span>2)  </span><span>if(undefined == x) </span><span>works  
</span><span>3) Mr. Presenter Guy is not aware of any problems with (2).</span><span> </span>

<span>The first statement is demonstrably false because of that "have to" in there.  The </span><span>typeof</span><span> trick works, but "have to" implies incorrectly that it is the *<span>only</span>* thing that works.  This works just as well: </span><span>if (x === undefined) </span><span>(Note that **triple** equals in there.  More on that below.) </span>

<span></span> <span>The second statement is also demonstrably false, because it has "false positives".  Yes, this detects when a value is undefined, but… </span>

<span>var x = null;  
</span><span>print(x == undefined); </span>

<span>will evaluate as </span><span>true</span><span>, even though </span><span>x </span><span>is clearly defined\!  </span><span>undefined</span><span> and </span><span>null</span><span> compare as equal to each other; this does not determine if a variable is undefined, it determines if a variable is undefined **<span>or is defined as a value which the comparison operator considers as equal to undefined. </span>**</span>

<span>The third statement was true; Mr. Presenter Guy was probably not aware of any problems.  I asked the original questioner to point out to Mr. Presenter Guy that the </span><span>==</span><span> operator gives false positives, so hopefully the third statement is also now false. </span>

<span></span> <span>What the heck is up with the </span><span>===</span><span> operator?  Here's the thing: the </span><span>==</span><span> operator is very lenient when comparing data of different types.  </span>

<span></span> <span>print("1" == 1);  // true </span>

<span></span> <span> Which is often very useful.  But then one day (April 23rd, 1997, if you care) we added the <span>switch </span>statement to JScript and this question immediate arose: what the heck does this do? </span>

<span></span> <span>x = 1;  
</span><span>switch(x)  
</span><span>{  
</span><span>case "1": print("string"); break;  
</span><span>case 1: print("number"); break;  
</span><span>} </span>

<span></span> <span>This </span><span>switch</span><span> statement in JScript is NOT logically equivalent to </span>

<span></span> <span>if (x == "1") print("string");  
</span><span>else if (x == 1) print("number") </span>

<span></span> <span>because then this would print "string" when clearly the other case is by far the better match.  </span>

<span></span> <span>The </span><span>switch</span><span> statement requires that not only must the case and the argument compare as equal, but **<span>they must also be of the same type.</span>**  </span>

<span></span> <span>Given that, it would then seem really weird if you could construct a </span><span>switch</span><span> statement with these comparison semantics but could not construct the equivalent </span><span>if</span><span> statement.  Therefore we added the </span><span>===</span><span> operator, which has the same semantics as the comparator used in the </span><span>switch</span><span> statement. </span>

<span></span> <span>This now explains why this works for undefined variables. The </span><span>===</span><span> operator checks to see whether the arguments are of the same type; since there is only one member of the undefined type, there can be no false positives.</span> <span>\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* </span>

<div>

<span></span>

</div>

<span></span> <span>In other news, [I've managed to avoid the terrible fate that befalls boys who will not propose marriage](http://weblogs.asp.net/ericlippert/archive/2004/07/12/181265.aspx "http://weblogs.asp.net/ericlippert/archive/2004/07/12/181265.aspx").  No one will be exiling *<span>me</span>* to a desert island surrounded by sharp rocks and dropping hate-mail valentine cards from helicopters\!  My erstwhile lovely girlfriend Leah is now my charming fiancée Leah, and I couldn't be more pleased.  The plan so far is to get married next summer on the beaver-shark-infested shores of romantic Lake Huron, spend a week or so in romantic Holland, and then throw a huge yet elegant dance party back in romantic Seattle.</span>

<span>(UPDATE: Leah has just sent me an email saying that if anyone does exile me to a desert island, she will rescue me and then bake me that cake. That's good to know.)</span>

</div>

</div>

