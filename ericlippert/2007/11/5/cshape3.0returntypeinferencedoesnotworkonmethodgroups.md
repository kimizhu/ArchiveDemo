# C\# 3.0 Return Type Inference Does Not Work On Method Groups

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/5/2007 10:48:00 AM

-----

**UPDATE**:

A lot of people have linked to this article which is somewhat out of date now. This article describes why in C\# 3.0, return type inference does not work when the argument is a method group and the formal parameter is a generic delegate. As you can see, there was a lot of feedback on this article; as a result, we revisited that design decision for C\# 4.0. In C\# 4.0, **return type inference works on method group arguments when the method group can be associated unambiguously with a completely fixed set of argument types deduced from the delegate**. Once the argument types associated with the method group are known, then overload resolution can determine unambiguously which method in the method group is the one associated with the delegate formal parameter; we can then make a return type inference from the specific method to the delegate return type.

I return you now to the original article.

-----

Thanks all for your excellent feedback on my variance series. I am still reading and digesting the feedback and I shall take it to the language design committee later this month.

Returning to less hypothetical but equally complicated issues, today, what I thought would be a rather obscure issue with C\# 3.0 type inference which multiple people have reported to me since the last beta. Perhaps it is not as obscure as I thought\!

Consider this (contrived, but simplified from real code) example where we want to see if a collection of customers has every member from London or not. The LINQ-to-Objects library has an All extension method sequence operator which returns true if a predicate is true of all members of a collection, so we can use that:

 

static string GetCity(Customer c)  
{  
  return c.City;  
}  
static bool IsLondon(string s)  
{  
  return s == "London";  
}

This works just fine:

 

var cities = customers.Select(c=\>GetCity(c));  
bool allLondon = cities.All(s=\>IsLondon(s));

Now, assuming that this is LINQ-to-Objects and those lambdas are going to be turned into delegates, why do we need the lambdas at all? We can just turn the method groups into delegates directly. This works just fine too:

 

var cities = customers.Select(c=\>GetCity(c));  
bool allLondon = cities.All(IsLondon);

But this does not work:

 

var cities = customers.Select(GetCity); // Inference error

That fails with a type inference error stating that the type arguments for Select could not be determined. If you supply them explicitly, everything works:

 

var cities = customers.Select\<Customer, string\>(GetCity); // No problem.

Why does this fail on the Select but not on the All? And is that the correct behaviour according to the specification?

Unfortunately, the published 3.0 specification states that this should work for the Select. I did not realize that the implementation was out of line with the specification until the spec was already on the web, and upon review, we realized that **the implementation was correct and the specification was wrong**. (We will batch up the specification errors we’ve discovered so far and fix them all at once in a future revision; we do not want to fix specification errors piecemeal.)

Let’s go through the process of type inference and we’ll see where things go wrong. The definition of Select is:

 

static IEnumerable\<R\> Select\<C, R\>(this IEnumerable\<C\> collection, Func\<C, R\> projection) { …

When we see

 

customers.Select(c=\>GetCity(c));

the type inference process reasons as follows. From the first argument we unambiguously deduce that C is Customer. That means that the second argument must be a Func\<Customer, R\>, but we do not know what R is. We then analyze the lambda as if it were written (Customer c)=\>{ return GetCity(c); }. We do overload resolution on GetCity(c) and make an exact match to the string GetCity(Customer) method. We therefore deduce that the lambda returns a string, and therefore deduce that R is string.

At this point we have now deduced both C and R. We verify that the arguments are all compatible with the deduced types of the formal parameters and life is good.

Now consider what happens when we see:

 

customers.Select(GetCity);

This time, type inference goes like this: again, we unambiguously deduce that C is Customer. The second argument is a Func\<Customer, R\>, but we do not know what R is. We have a method group as the second argument. In order to determine what method the method group refers to, we must do overload resolution on it. But… we do not have any arguments\! How can we do overload resolution on GetCity if we do not know what the arguments to the call are? Sure, in this case there is only one method in the method group and it is not generic, so we could just pick it. But we ought to solve the general problem here.

Now, you might point out that we manage to do overload resolution correctly for this case:

 

Func\<Customer, string\> f = GetCity;

Overload resolution works just fine there even though we have no arguments to GetCity. In that case we take the signature of the delegate and use the type of each parameter in the delegate signature to do overload resolution. That is, we do it as though we were doing overload resolution on GetCity(default(Customer)). That is then enough information to do overload resolution on GetCity.

I seem to be digging myself in deeper. Why don’t we do the same thing in the type inference case above?

Because I skipped a step in overload resolution. What if we need to do type inference on GetCity?

Suppose instead of the simple example above we had something like

 

S GetCity\<X, S\>(X x) { …

Now consider what happens when we have

 

Func\<Customer, string\> f = GetCity;

In this case before we do overload resolution we first do type inference FROM Func\<Customer, string\> TO GetCity\<X, S\> and deduce that X and S are Customer and string. Then overload resolution proceeds normally.

Am I still digging myself deeper? No. Perhaps now you see the problem. Suppose we had a generic GetCity and we said:

 

customers.Select(GetCity);

We get as far as deducing that we have Func\<Customer, R\>, and now we are in a hideous bind. In order to determine R we need to determine the return type of GetCity. To determine the return type of GetCity we need to do overload resolution on the method group. To do overload resolution on the method group, we need to do type inference on the method group. To do type inference on it we need to know what delegate type it is going to. It is going to Func\<Customer, R\>. But R is what we are trying to figure out\! **We do not yet know the delegate type that it is going to, so overload resolution is in general impossible**. This is a circle that will stay unbroken (by and by Lord, by and by.)

It does not actually make any sense to try to do overload resolution in the general type inference scenario. We could have come up with a weaker form of overload resolution that worked in an environment where type inference might not succeed, but that would then introduce a brand new and subtly incompatible set of overload resolution rules which would only come into effect in these obscure inference scenarios where one type inference triggers another. We want the language semantics to remain understandable and testable, and we’re pushing that edge pretty hard already with the new type inference features.

**Therefore this scenario is illegal; you cannot do return type inference on a method group, only on a lambda.** The compiler can peer into the lambda and determine what type it *would* return were all its arguments known, but the compiler cannot do that with a method group. There is simply less information available with a method group.

Why then does the All(IsLondon) case work?

Because the signature of All is

 

static bool All\<C\>(this IEnumerable\<C\> collection, Func\<C, bool\> predicate)

By the type we need to do overload resolution on the method group the type of the second parameter is *entirely* known. The rule that return type inference cannot be done on method groups does not apply because there is no return type inference necessary.

\[UPDATE: In the C\# specification, an expression naming a method is called a "method group". Unfortunately, in the compiler source code they are called "member groups", and this gets me in the bad habit of using imprecise language. In the original version of this article I referred to "member groups" throughout; I've fixed those up. Sorry for any confusion.\]

