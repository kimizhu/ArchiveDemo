# JScript Equality Operators, plus More On Mad Crushes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/26/2004 12:01:00 PM

-----

First, the technical stuff.  Here's a recent question I received about missing data in JScript.   

> I’ve seen it stated that in order to determine if a variable is undefined, you have to use if("undefined" == typeof(x))  
> I attended a presentation on JScript today where the presenter used if(undefined == x)  
> When I asked about that, he stated that it works and he is not aware of any problems with it. Is he correct?

I discussed related issues earlier ([here](http://weblogs.asp.net/ericlippert/archive/2003/09/30/53120.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/30/53120.aspx") and [here](http://weblogs.asp.net/ericlippert/archive/2003/10/01/53128.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/01/53128.aspx") and [here](http://weblogs.asp.net/ericlippert/archive/2003/11/05/53336.aspx "http://weblogs.asp.net/ericlippert/archive/2003/11/05/53336.aspx")), but I figure that this is another opportunity for clarification. 

 There are three statements made.  

1\) In order to determine if a variable is undefined, you **have to** use if("undefined" == typeof(x))  
2)  if(undefined == x) works  
3) Mr. Presenter Guy is not aware of any problems with (2). 

The first statement is demonstrably false because of that "have to" in there.  The typeof trick works, but "have to" implies incorrectly that it is the *only* thing that works.  This works just as well: if (x === undefined) (Note that **triple** equals in there.  More on that below.) 

 The second statement is also demonstrably false, because it has "false positives".  Yes, this detects when a value is undefined, but… 

var x = null;  
print(x == undefined); 

will evaluate as true, even though x is clearly defined\!  undefined and null compare as equal to each other; this does not determine if a variable is undefined, it determines if a variable is undefined **or is defined as a value which the comparison operator considers as equal to undefined. **

The third statement was true; Mr. Presenter Guy was probably not aware of any problems.  I asked the original questioner to point out to Mr. Presenter Guy that the == operator gives false positives, so hopefully the third statement is also now false. 

 What the heck is up with the === operator?  Here's the thing: the == operator is very lenient when comparing data of different types.  

 print("1" == 1);  // true 

  Which is often very useful.  But then one day (April 23rd, 1997, if you care) we added the switch statement to JScript and this question immediate arose: what the heck does this do? 

 x = 1;  
switch(x)  
{  
case "1": print("string"); break;  
case 1: print("number"); break;  
} 

 This switch statement in JScript is NOT logically equivalent to 

 if (x == "1") print("string");  
else if (x == 1) print("number") 

 because then this would print "string" when clearly the other case is by far the better match.  

 The switch statement requires that not only must the case and the argument compare as equal, but **they must also be of the same type.**  

 Given that, it would then seem really weird if you could construct a switch statement with these comparison semantics but could not construct the equivalent if statement.  Therefore we added the === operator, which has the same semantics as the comparator used in the switch statement. 

 This now explains why this works for undefined variables. The === operator checks to see whether the arguments are of the same type; since there is only one member of the undefined type, there can be no false positives. \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 

 In other news, [I've managed to avoid the terrible fate that befalls boys who will not propose marriage](http://weblogs.asp.net/ericlippert/archive/2004/07/12/181265.aspx "http://weblogs.asp.net/ericlippert/archive/2004/07/12/181265.aspx").  No one will be exiling *me* to a desert island surrounded by sharp rocks and dropping hate-mail valentine cards from helicopters\!  My erstwhile lovely girlfriend Leah is now my charming fiancée Leah, and I couldn't be more pleased.  The plan so far is to get married next summer on the beaver-shark-infested shores of romantic Lake Huron, spend a week or so in romantic Holland, and then throw a huge yet elegant dance party back in romantic Seattle.

(UPDATE: Leah has just sent me an email saying that if anyone does exile me to a desert island, she will rescue me and then bake me that cake. That's good to know.)

