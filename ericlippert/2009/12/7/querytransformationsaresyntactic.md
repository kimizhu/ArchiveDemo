# Query transformations are syntactic

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/7/2009 6:20:00 AM

-----

As you probably know, there are two ways to write a LINQ query in C\#. The way I personally prefer is to use the “query comprehension” syntax:

 

from customer in customerList  
where customer.City == "London"  
select customer.Name

Or you can, equivalently, use the “fluent method call” syntax:

 

customerList  
.Where(customer=\>customer.City == "London")  
.Select(customer=\>customer.Name)

These are guaranteed to be equivalent because **the compiler simply transforms the former syntax into the latter syntax before it compiles it**. An interesting aspect of this transformation is that it is (almost) entirely **syntactic**. (The "transparent identifiers" generated for certain queries require a small amount of semantic analysis to determine the corresponding anonymous type, but for the most part, all we do is just pull the raw hunks of code out of each clause and reorganize the program into the method call form.) Once it is in a form that the rest of the compiler can understand, then semantic analysis proceeds as usual.

This means that it is perfectly legal to do dumb things. For example, suppose we decide that by “where” we mean don’t mean “filter”, we actually mean “multiply”. And by “select” we actually mean “add”, not “project”. No problem\!

 

static class ExtInt  
{  
    public static int Where(this int c, Func\<int, int\> f)  
    {  
        return f(0) \* c;  
    }  
    public static int Select(this int c, Func\<int, int\> f)  
    {  
        return f(0) + c;  
    }  
}  
…  
int ten = 10;  
int twenty = 20;  
int thirty = 30;   
int result = from c in ten where twenty select thirty;  
Console.WriteLine(result);

And sure enough, ten where/times twenty select/plus thirty is 230. This is a deeply strange way to write a mathematical expression, but legal. The semantics of the Where method are supposed to be “takes a collection of T and a predicate mapping T to bool, and returns a filtered collection of those T items which match the predicate”. This method does not take a collection or a predicate, it does not return a collection, and it certainly does not have the semantics of filtering, but the compiler neither knows nor cares about any of those facts. All the compiler does is syntactically turn that line into  

int result = ten.Where(c=\>twenty).Select(c=\>thirty);

and compile it; *that* line compiles just fine, so, no problem as far as the compiler is concerned.

The C\# specification has a section which describes the pattern we expect a query provider to implement: that Where takes a predicate, and so on. But we perform no checks whatsoever that you successfully implemented a query provider that implements either the form or the semantics of our recommended pattern. If you do something crazy, like redefine “where” to take something other than a predicate and you get crazy results, then, well, what can I tell you? If it hurts when you do that then don’t do that\!

