<div id="page">

# Type inference woes, part four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/18/2006 1:30:00 PM

-----

<div id="content">

<div class="mine">

[Last time in this series](http://blogs.msdn.com/ericlippert/archive/2006/05/30/610769.aspx) I discussed how we are probably going to identify a "best" type from a set of expressions. Clearly we need to solve this problem for features such as implicitly typed arrays. We are also considering extending this algorithm to affect type inference of generic methods.

As always, the reason we're considering this is emminently practical; we try hard to not add new language semantics just for the heck of it, but rather in response to specific user problems. Before I get into the motivating problem, let me describe a simpler case that might look familiar:

<span class="code">void M\<T\>(T t1, T t2){}  
...  
M(myShort, 0); </span>

In our currently shipped C\# compiler, the type inference algorithm insists that all inferences be 100% completely consistent. <span class="code">T </span>cannot be both <span class="code">int</span> and <span class="code">short</span>, so inference fails in this case.

It might be kind of nice to say, you know what, the "best type" algorithm would tell us that <span class="code">short</span> is best, so why don't we use it?

Of course, there are some situations where that stops working. We'd still need this to fail, for instance:

<span class="code">void M\<T\>(List\<T\> t1, List\<T\> t2){}  
...  
M(myListOfShorts, myListOfInts); </span>

Generic types are neither covariant nor contravariant, so neither choice works in this case. However, it *does* work in this case:

<span class="code">void M\<T\>(Func\<T\> t1, Func\<T\> t2){}  
...  
M(()=\>myInt, ()=\>myShort); </span>

because a lambda which returns a <span class="code">short</span> *is* convertible to a delegate which returns an <span class="code">int</span>, so inference to <span class="code">int</span> could succeed.

The motivating example for all of this is the join scenario. For those of you who are not database wonks, joining is one of the fundamental operations on tabular data. You typically have two tables with some relationship between them. For instance, you might have a list of customers, where each customer has a customer id, and a list of orders where each order has an associated customer id. You "join" the tables by producing a new table of records associating each customer with their orders. Maybe the new table is the customer name and the amount of each of their orders.

To represent a generic join operation as a method we need two tables of records, a key extractor for each table, and a projector from the joined records to the output records. The caller might look something like:

<span class="code">var results = Join(  
    myCustomers,  
    myOrders,  
    customer=\>customer.Id,  
    order=\>order.CustomerId,  
    (customer, order)=\>new {customer.Name, order.Price});  
</span>

The fully generic method might look something like:

<span class="code">IEnumerable\<RES\> Join\<REC1, REC2, KEY, RES\>(  
    IEnumerable\<REC1\> rec1s,  
    IEnumerable\<REC2\> rec2s,  
    Func\<REC1, KEY\> rec1KeyExtractor,  
    Func\<REC2, KEY\> rec2KeyExtractor,  
    Func\<REC1, REC2, RES\> projector) { ... }  
</span>

So now perhaps you see the problem. What if the customer key type is <span class="code">int </span>but the order table's customer key type is <span class="code">Nullable\<int\> </span>? We would like to infer that <span class="code">Nullable\<int\> </span>is the better choice for <span class="code">KEY </span>but today the type inference algorithm insists on 100% consistency.

As you can see, this is rapidly becoming complex. We would like to come up with an inference algorithm that solves this problem, but at the same time can be clearly described in the standard and understood by mortals. (Where by "understood by mortals" I mean "such that Eric has some chance of implementing the algorithm correctly"\!) Any thoughts or comments you might have would be appreciated; the language design committee will be debating these issues this week and so this is the time to give feedback about these things.

</div>

</div>

</div>

