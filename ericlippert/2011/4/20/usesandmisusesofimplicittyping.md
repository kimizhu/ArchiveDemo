# Uses and misuses of implicit typing

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/20/2011 10:37:52 AM

-----

One of the most controversial features we've ever added was *implicitly typed local variables*, aka "var". Even now, years later, I still see [articles](http://www.brad-smith.info/blog/archives/336) debating the [pros](http://devlicio.us/blogs/hadi_hariri/archive/2009/11/20/var-improves-readability.aspx) and [cons](http://weblogs.asp.net/stevewellens/archive/2009/11/19/can-the-c-var-keyword-be-misused.aspx) of the feature. I'm often asked what my opinion is, so here you go.

Let's first establish what the **purpose** of code is in the first place. For this article, the purpose of code is **to create value by solving a business problem**.

Now, sure, that's not the purpose of all code. The purpose of the assembler I wrote for my CS 242 assignment all those years ago was not to solve any business problem; rather, its purpose was to teach me how assemblers work; it was pedagogic code. The purpose of the code I write to solve [Project Euler](http://projecteuler.net/) problems is not to solve any business problem; it's for my own enjoyment. I'm sure there are people who write code just for the aesthetic experience, as an art form. There are lots of reasons to write code, but for the sake of this article I'm going to make the reasonable assumption that people who want to know whether they should use "var" or not are asking in their capacity as professional programmers working on complex business problems on large teams.

Note that by "business" problems I don't necessarily mean *accounting* problems; if analyzing the human genome in exchange for National Science Foundation grant money is your business, then writing software to recognize strings in a sequence is solving a business problem. If making a fun video game, giving it away for free and selling ads around it is your business, then making the aliens blow up convincingly is solving a business problem. And so on. I'm not putting any limits here on what sort of software solves business problems, or what the business model is.

Second, let's establish what decision we're talking about here. The decision we are talking about is whether it is better to write a local variable declaration as:

 

TheType theVariable = theInitializer;

or

 

var theVariable = theInitializer;

where "TheType" is the compile-time type of theInitializer. That is, I am interested in the question of whether to use "var" *in scenarios where doing so does not introduce a semantic change*. I am explicitly not interested in the question of whether

 

IFoo myFoo = new FooStruct();

is better or worse than

 

var myFoo = new FooStruct();

because those two statements do different things, so it is not a fair comparison. Similarly I am not interested in discussing bizarre and unlikely corner cases like "what if there is a struct named var in scope?" and so on.

In this same vein, I'm interested in discussing the pros and cons *when there is a choice*. If you have already decided to use anonymous types then the choice of whether to use implicit typing has already been made:

 

var query = from c in customers select new {c.Name, c.Age};

The question of whether it is better to use nominal or anonymous types is a separate discussion; if you've decided that anonymous types are worthwhile then you are almost certainly going to be using "var" because there is no good alternative.

Given that the overarching purpose is assumed to be solving business problems, what makes good code? Obviously that's a huge topic but three relevant factors come to mind. Good code:

  - works correctly according to its specification to actually solve the stated problem
  - communicates its meaning to the reader who needs to understand its operation
  - allows for relatively low-cost modification to solve new problems as the business environment changes.

In evaluating whether or not to use "var" we can dismiss the first concern; I'm only interested in pros and cons of cases where using var does not change the meaning of a program, only its textual representation. If we change the text without changing its semantics then by definition we have not changed its correctness. Similarly, using or not using "var" does not change other observable characteristics of the program, such as its performance. The question of whether or not to use "var" hinges upon its effect on the human readers and maintainers of the code, not on its effect upon the compiled artefact.

What then is the effect of this abstraction on the reader of the code?

All code is of course an abstraction; that's the whole reason why we have high-level languages rather than getting out our voltmeters and programming at the circuit level. Code abstractions necessarily emphasize some aspects of the solution while "abstracting away" other aspects. A good abstraction hides what is irrelevant and makes salient what is important. You might know that on x86 chips C\# code will typically put the value returned by a method in EAX and typically put the "this" reference in ECX, but you don't need to know any of that to write C\# programs; that fact has been abstracted away completely.

It is clearly not the case that more information in the code is always better. Consider that query I referred to earlier. What is easier to understand, that query, or to choose to use a nominal type and no query comprehension:

 

IEnumerable\<NameAndAge\> query = Enumerable.Select\<Customers, NameAndAge\>(customers, NameAndAgeExtractor);

along with the implementations of the NameAndAge class, and the NameAndAgeExtractor method? Clearly the query syntax is much more abstract and hides a lot of irrelevant or redundant information, while emphasizing what we wish to be the salient details: that we are creating a query which selects the name and age of a table of customers. **The query emphasizes the business purpose of the code; the expansion of the query emphasizes the mechanisms used to implement that purpose.**

The question then of whether "var" makes code better or worse for the *reader* comes down to two linked questions:

1\) is ensuring salience of the variable's type important to the understanding of the code? and,  
2\) if yes, is stating the type in the declaration necessary to ensure salience?

Let's consider the first question first. Under what circumstances is it necessary for a variable's type to be clearly understood when reading the code? Only when the **mechanism** of the code -- the "how it works" -- is more important to the reader than the **semantics** -- the "what its for".

In a high-level language used to solve business problems, I like the mechanisms to be abstracted away and the salient features of the code to be the business domain logic. That's not always the case of course; sometimes you really do care that this thing is a uint, it has got to be a uint, we are taking advantage of the fact that it is a uint, and if we turned it into a ulong or a short, or whatever, then the mechanism would break.

For example, suppose you did something like this:

 

var distributionLists = MyEmailStore.Contacts(ContactKind.DistributionList);

Suppose the elided type is DataTable. **Is it important to the reader to know that this is a DataTable?** That's the key question. Maybe it is. Maybe the correctness and understandability of the rest of the method depends completely on the reader understanding that distributionLists is a DataTable, and not a List\<Contact\> or an IQueryable\<Contact\> or something else.

But hopefully it is not. Hopefully the rest of the method is perfectly understandable with only the semantic understanding, that distributionLists represents a collection of distribution lists fetched from a storage containing email contacts.

Now, to the crowd who says that of course it is better to always know the type, because knowing the type is important for the reader, I would ask a pointed question. Consider this code:

 

decimal rate = 0.0525m;  
decimal principal = 200000.00m;  
decimal annualFees = 100.00m;  
decimal closingCosts = 1000.00m;  
decimal firstPayment = principal \* (rate / 12) + annualFees / 12 + closingCosts;

Let's suppose that you believe that it is important for all the types to be stated so that the code is more understandable. Why then is it not important for the types of all those subexpressions to be stated? There are at least four subexpressions in that last statement where the types are not stated. **If it is important for the reader to know that 'rate' is of type decimal, then why is it not also important for them to know that (rate / 12) is of type decimal, and not, say, int or double?**

The simple fact is that the compiler does huge amounts of type analysis on your behalf already, types which never appear in the source code, because for the most part those types would be distracting noise rather than helpful information. **Sometimes the declared type of a variable is distracting noise too.**

Now consider the second question. Suppose for the sake of argument it is necessary for the reader to understand the storage type. Is it necessary to state it? Often it is not:

 

var prices = new Dictionary\<string, List\<decimal\>\>();

It might be necessary for the reader to understand that prices is a dictionary mapping strings to lists of decimals, but that does not mean that you have to say

 

Dictionary\<string, List\<decimal\>\> prices = new Dictionary\<string, List\<decimal\>\>();

Clearly use of "var" does not preclude that understanding.

So far I've been talking about reading code. What about maintaining code? Again, var can sometimes hurt maintainability and sometimes help it. I have many times written code something like:

 

var attributes = ParseAttributeList();  
foreach(var attribute in attributes)  
{  
    if (attribute.ShortName == "Obsolete") ...

Now suppose I, maintaining this code, change ParseAttributeList to return a ReadOnlyCollection\<AttributeSyntax\> instead of List\<AttributeSyntax\>. With "var" I don't have to change anything else; all the code that used to work still works. Using implicitly typed variables helps make refactorings that do not change semantics succeed with minimal edits. (And if refactoring changes semantics, then you'll have to edit the code's consumers regardless of whether you used var or not.)

Sometimes critics of implicitly typed locals come up with elaborate scenarios in which it allegedly becomes difficult to understand and maintain code:

var square = new Shape();  
var round = new Hole();  
... hundreds of lines later ...  
bool b = CanIPutThisPegInThisHole(square, round); // Works\!

Which then later gets "refactored" to:

 

var square = new BandLeader("Lawrence Welk");  
var round = new Ammunition();  
... hundreds of lines later ...  
bool b = CanIPutThisPegInThisHole(square, round); // Fails\!

In practice these sorts of contrived situations do not arise often, and the problem is actually more due to the bad naming conventions and unlikely mixtures of business domains than the lack of explicit typing.

Summing up, my advice is:

  - Use var when you have to; when you are using anonymous types.
  - Use var when the type of the declaration is obvious from the initializer, especially if it is an object creation. This eliminates redundancy.
  - Consider using var if the code emphasizes the semantic "business purpose" of the variable and downplays the "mechanical" details of its storage.
  - Use explicit types if doing so is *necessary* for the code to be correctly understood and maintained.
  - Use descriptive variable names regardless of whether you use "var". Variable names should represent the semantics of the variable, not details of its storage; "decimalRate" is bad; "interestRate" is good.

