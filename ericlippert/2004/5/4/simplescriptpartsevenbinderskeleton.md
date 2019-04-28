# SimpleScript Part Seven: Binder Skeleton

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 10:51:00 AM

-----

 

In [Part Five](/ericlippert/archive/2004/04/22/118585.aspx "http://blogs.msdn.com/ericlippert/archive/2004/04/22/118585.aspx") I was discussing modules: there is a "global module" and any number of additional modules.  Each module is associated with a named item, and the only module which is associated with more than one named item is the global module.  This means that each module is going to need its own **name table** to keep track of the functions, the variable names and values, etc, in it.  We'll call such devices "binders" because they bind names to values. 

For reasons which will become more clear in future episodes, it is convenient to have the binder implement the IDispatch interface.  Normally if you had to implement IDispatch you'd build a type library, and then have your implementation of IDispatch call through to the ITypeInfo methods which look up dispatch ids, invoke methods, and so on.  There's a good reason for that: rolling your own IDispatch code can be extremely tricky.  But, we're pretty much stuck with it here -- we haven't got the convenience of a known vtable interface to wrap a type library around at compile time.  The script engine name tables are going to be extremely dynamic, so we're going to have to roll our own IDispatch. 

The code isn't nearly done yet, obviously, but I've got a good skeleton in place.  Take a look at [binder.cpp](/ericlippert/articles/125805.aspx "http://blogs.msdn.com/ericlippert/articles/125805.aspx") for the details.   I've pretty much got the high-level semantics of the dispatch interface worked out, but the actual low-level implementation details all return E\_NOTIMPL.  What I need to build here is a special hash table which can efficiently look up values by both numeric id and name.  That's not too challenging; what is going to be more difficult is implementing the SCRIPTITEM\_GLOBALMEMBERS feature.  We need to be able to add another arbitrary dispatch object to our binder and dynamically aggregate all of its methods, without causing any collision in dispatch ids\!  

Default property semantics are a little weird.  In the wacky world of late bound objects, an object may have a "default" method.  This feature was created so that you could call the Item method of a collection the same way you dereference an array in Visual Basic -- you make it look like a function call.  That is, since this code works: 

Dim arr(10)  
arr(1) = 123 

then so should this code 

dict = CreateObject("Scripting.Dictionary")  
dict(1) = 123 

the last line is the same as 

dict.item(1) = 123 

Such is the wackiness of VB -- because the array dereference and function call syntax are conflated, they ended up making *function calls on invisible methods be legal on the left hand side of an assignment statement.*  The mind fairly boggles.   We'll be good citizens in SimpleScript.  If we have an object in our binder and someone attempts to call it, we'll just pass on the arguments to its default method and let it sort out what the right thing to do is.  

As you can see, there is a whole lot of parameter validation, and altogether about eleven separate cases that I cover in the implementation of Invoke.  The actual implementation in VBScript and JScript is much, much more complex than this because they support IDispatchEx, garbage collection, property accessor functions, and numerous other features that complicate the code.  But still, this is the longest function you're likely to see me write in this project; I'll be very surprised if I write any code more complicated than the dispatch logic. 

Speaking of garbage collection, I'm not quite sure what kind of garbage collector I'm going to build into SimpleScript.  Building a mark-and-sweep collector like JScript has would be instructive, but it would also take a lot of time and effort.  What do you guys think?  (If you look at [invoke.cpp](/ericlippert/articles/125807.aspx "http://blogs.msdn.com/ericlippert/articles/125807.aspx") you'll see that I'm thinking ahead about issues we're going to run into in garbage collection.  We're going to run into related issues in the script execution code; it is possible for ill-behaved hosts to shut down the script engine in a callback while it is executing\!  We need to be robust in the face of such behaviour.) 

The next step is to get the actual guts of the binder working, and then build a list of code blocks to be executed when the script engine goes to started state.  The clone semantics for the code blocks are a little tricky, so we'll get that straightened away, and then maybe, just maybe, actually write a language parser.

UPDATE:  For your convenience, I've put a zip file containing all the sources at <http://www.eric.lippert.com/simplescript.zip>.  I'll keep this zip file updated as I change the sources on the blog.

