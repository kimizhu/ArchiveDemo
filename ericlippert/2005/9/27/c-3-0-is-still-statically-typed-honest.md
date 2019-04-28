<div id="page">

# C\# 3.0 is still statically typed, honest\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/27/2005 1:03:00 PM

-----

<div id="content">

Since LINQ was announced I've seen a lot of really positive feedback and a lot of questions and concerns. Keep 'em coming\! We need this feedback so that we can both correct misconceptions early and get the design right now. One of the misconceptions that I've seen a lot over the last few days in forums, blog posts and private emails is a confusion about what the new "type inferencing" feature implies for the type safety of the language. Apparently we have not been sufficiently clear on this point: **C\# 3.0 will be statically typed, just like C\# 1.0 and 2.0. The var declaration style does not introduce dynamic typing or duck typing to C\#.** I think the confusion may arise from familiarity with other languages such as JScript. In JScript this is perfectly legal: var foo = new Blah();  
foo = 123;  
foo = "hello"; JScript is a dynamically typed language. You can assign any value of any type to a var. In C\# 3.0, the var statement means "look at the type of the thing assigned to the variable, and act as though the variable was declared with that type." In other words, in C\# the code above is just a syntactic sugar for **Blah** foo = new Blah();  
foo = 123;  
foo = "hello"; which of course would produce a type error on the second and third lines. If you take a look at section 26.1 of the C\# 3.0 specification you'll see that the var statement has a lot of restrictions on it to ensure that the compiler always has enough information to make the correct type inference. Namely:

  - the declarator must include an initializer, so that we can infer the type of the variable from the type of the initializer
  - the initializer has to be something that we can figure out the type of – not null or a collection initializer

Compare this to JScript .NET, which has a much stronger type inference mechanism. JScript .NET does not require initializers in var statements; the compiler tracks all assignments to the variable and infers the best type. If, say, only strings are assigned to a variable then it will infer the string type. JScript .NET also infers return types of functions by a similar mechanism. But the goal of the JScript .NET type inference mechanism was to **increase the performance of legacy dynamically typed code**. If we can infer a type and thereby generate faster, smaller code, we do so.  If not, we don't. Then why introduce this syntactic sugar in C\# 3.0? C\# doesn't have a body of legacy dynamic code like JScript and already generates efficient code. There are two reasons, one which exists today, one which will crop up in 3.0. The first reason is that this code is incredibly ugly because of all the redundancy: Dictionary\<string, List\<int\>\> mylists = new Dictionary\<string, List\<int\>\>(); And that's a simple example – I've written worse. Any time you're forced to type exactly the same thing twice, that's a redundancy that we can remove. Much nicer to write var mylists = new Dictionary\<string, List\<int\>\>(); and let the compiler figure out what the type is based on the assignment. Second, C\# 3.0 introduces anonymous types. Since anonymous types by definition have no names, you need to be able to infer the type of the variable from the initializing expression if its type is anonymous. We'll discuss the reasoning behind anonymous types in another post.

</div>

</div>

