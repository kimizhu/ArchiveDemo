# What's the Difference, Part Four: into vs into

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/31/2009 9:30:00 AM

-----

The keyword "into" in a query comprehension means two different things, depending on whether it follows a join or select/group. If it follows a join, it turns a join into a group join. If it follows a select or group then it introduces a query continuation. These two features are quite different, but easily confused.

First, the group join. Suppose you've got a key -- a customer id number -- that is used as the primary key of a collection of customers, and as a foreign key of a collection of credit card numbers. That is, you have a class Customer with fields Id, Name, Address, and so on, and a class CreditCard with fields CustomerId, CardType, Number, and so on. Suppose customer Bob has a Visa and a Discover, and customer Alice has a Visa and a Mastercard. So we have the customer data:

 

101, Bob  
102, Alice

and the credit card data:

 

101, Visa  
101, Discover  
102, Visa  
102, MasterCard

If we create a query

 

from customer in customers  
join card in cards on customer.Id equals card.CustomerId  
select new {customer.Name, card.Kind}

The results of the query would be 

 

Bob, Visa  
Bob, Discover  
Alice, Visa  
Alice, Mastercard

right? This is just a straightforward join. We end up with a list with four items on it. But this is probably not what you actually want in this case. Suppose you wanted a list of customers, and with each customer on the list, a list of their credit cards. You can use a group join:

from customer in customers  
join card in cards on customer.Id equals card.CustomerId into cardList  
select new {customer.Name, Cards = cardList} 

The results of this query would be two records, not four:

 

Bob, { Visa, Discover }  
Alice, { Visa, Mastercard }

Basically, the "into" in a group join is the portion of the query expression which logically gathers up the results of all the joined records, makes them into a sequence, and stuffs them into temporary variable cardList.

A query continuation means something rather different. The point of a query continuation is to make it easy to "pipe" the results of one query into the next. For example, suppose you want to find all the brown-eyed children of who have at least one blue-eyed sibling. The obvious way to do that would be something like

 

from parent in parents  
from child in parent.Children  
where child.EyeColor == "Brown"  
where parent.Children.Any(c=\>c.EyeColor == Blue)  
select child

but let's suppose that we don't want to do that. Suppose there are a lot of large families of brown-eyed children with no blue-eyed siblings; a naive search could be quite inefficient. You might think to narrow it down the other way first. That is, it might be faster to find all the parents who have a blue-eyed child, and then extract from that small set of parents all the brown-eyed children. That's easiest to do in two queries. First find the parents, then from that build a second query which projects out the children:

 

var parentsWithABlueEyedChild =  
    from parent in parents  
    where parent.Children.Any(c=\>c.EyeColor == Blue)  
    select parent;  
var brownEyedChildren =  
    from p in parentsWithABlueEyedChild  
    from child in p.Children  
    where child.EyeColor == Brown  
    select child;

Now, we could combine this into one big query easily enough:

 

var brownEyedChildren =  
    from p in (  
        from parent in parents  
        where parent.Children.Any(c=\>c.EyeColor == Blue)  
        select parent)  
    from child in p.Children  
    where child.EyeColor == Brown  
    select child;

But... yuck. Imagine this nesting a few deeper. It makes a total mess. Notice how we introduce range variable "p" first, and then we have to get through a whole other query before it becomes relevant again. We're introducing range variables in the "inside out" order here. A query continuation simply allows you to put this order "right-side in" again, by moving the "p" range variable to after the initial query:

 

var brownEyedChildren =   
    from parent in parents   
    where parent.Children.Any(c=\>c.EyeColor == Blue)   
    select parent into p  
    from child in p.Children  
    where child.EyeColor == Brown  
    select child;

Notice that in the group join case, you can think of the "into" identifier as being something logically representing *the sequence* that results from the group of matching joined items. But in the query continuation case, the "into" identifier does not represent *the sequence* from the first query -- the query itself is an object which represents the first sequence\! Rather, "p" represents a *range variable that picks one member at a time out of the sequence*. Remember, the "into" in a query continuation is just a fancy way of saying from p in (blah); the "p" is a *range variable* that ranges over the sequence of elements of (blah), but it is not the sequence of elements of (blah) itself.

