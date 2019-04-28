# Method Type Inference Changes, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/17/2008 11:20:00 AM

-----

I want to start this by discussing the purpose of method type inference, and clearing up some potential misunderstandings about type inference errors.

First off though, a brief note on nomenclature. Throughout this series when I say "type inference" I mean "method type inference", not any of the other forms of type inference we have in C\# 3.0. (Implicitly typed locals, implicitly typed arrays, and so on.)

The purpose of type inference is to allow generic methods to be called without explicitly specifying the generic types. For example, if you have a generic method

 

static T Largest\<T\>(IEnumerable\<T\> list) where T : IComparable\<T\> { ... }

then it would be nice to be able to say

 

decimal maxPrice = Largest(prices);

without having to say redundant information:

 

decimal maxPrice = Largest\<decimal\>(prices);

This was useful in C\# 2.0, but with the advent of extension methods and query comprehensions in C\# 3.0, it is downright vital. If we made Largest into an extension method then clearly you want to be able to say

 

from customer in customers  
from invoices in customers.invoices  
select invoices.prices.Largest();

and not have to insert this ugly mechanism information into your lovely semantic query:

 

from customer in customers  
from invoices in customers.invoices  
select invoices.prices.Largest\<decimal\>();

When you call a method without an explicit argument list, overload resolution must figure out which method you meant to call. The way type inference works with overload resolution is that all methods of that name are put into a pool called the candidate set. If any of them are generic, we run a type inference algorithm to see if the generic type arguments can be inferred for the method. If they can, then the constructed generic method stays in the candidate set; if inference fails then it is removed. Then all methods which are inapplicable -- that is, the arguments cannot be converted to the parameter types -- are removed. Of the remaining applicable candidates, either a unique best candidate is chosen. If a unique best candidate cannot be identified then overload resolution fails.

Note that type inference failure is not actually an error in of itself. However, type inference failure (or success\!) might cause real errors downstream in a number of ways. For example:

  - if type inference fails and the candidate set becomes empty as a result, then overload resolution fails
  - if type inference fails when the user intended the generic method to be the chosen one, and the remaining candidates have no unique best member, then overload resolution fails
  - if type inference succeeds when it wasn't expected to then there might be more methods in the candidate set than expected. If the inferred method ties for "bestness" with another candidate then overload resolution fails. (The tiebreaker rules in overload resolution are designed to avoid this scenario, but it is still possible in contrived situations.)

This then brings up the question of what error message to display when overload resolution fails. In C\# 2.0 we have a heuristic which tries to detect when overload resolution failed because of the first case (which is the most common of the three cases.) In those cases it gives an error message that says "type inference failed" rather than "overload resolution failed". Even though the actual error as far as the language specification is concerned is that the candidate set contained no unique best applicable member, the user typically thinks of the cause of the error as being type inference failure in many cases, so we try to do a good job of reporting that.

It works pretty well in C\# 2.0, but we struggled with this a lot in C\# 3.0. Suppose you end up in a situation where you have a query:

 

IEnumerable\<Customer\> customers = ...;  
var firstNames = from c in customers select c.FristName;

Oops. That is going to be translated into customers.Select(c=\>c.FristName). There are no methods named "Select" on IEnumerable\<Customer\>, we look for extension methods and find Enumerable.Select\<T, R\>(this IEnumerable\<T\> list, Func\<T, R\> selector). Type inference infers T as Customer, but cannot figure out what R is because Customer does not have a property FristName. Therefore type inference fails, and therefore overload resolution fails, and therefore our heuristic takes over and gives the error that type inference failed on Select.

I think you would agree that the user does not think of the error in this program as being a failure of overload resolution or type inference; they think that the failure is that there's a typo in one clause of the query. Wes did quite a bit of work on the heuristics to go back one step further, and report why type inference failed; because the body of the lambda could not be successfully bound.

So that's one change to type inference between C\# 2.0 and C\# 3.0, but I seem to have gotten ahead of myself somewhat. Next time we'll take a closer look at how the mechanism of type inference works in C\# 2.0.

