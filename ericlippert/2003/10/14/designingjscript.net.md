# Designing JScript .NET

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/14/2003 7:44:00 PM

-----

A while back a reader asked for a rundown on some of the design decisions we made when designing JScript .NET.  That's a huge topic, but fortunately I started writing a book on the subject a few years ago that never found a publisher.  Tell you what -- whenever I can't think of something more interesting to post, I'll put snippets of it up on my blog.

 

 

There were four main design goals for JScript .NET:

 

 

1\)      JScript .NET should be **an extension of JScript**. This means that whenever possible a legal JScript program should be a legal JScript .NET program. JScript .NET must **continue to be usable as a dynamic scripting language**. Further, it should be easy to take an existing JScript Active Server Page and transform it into a JScript .NET page for ASP.NET.

 

 

2\)      JScript .NET should **afford the creation of highly performant programs**. In particular, JScript .NET should work well with ASP.NET because performance is often critical in server-side applications. Also, JScript .NET should warn programmers when they use a language feature which could adversely impact performance.

 

 

3\)      JScript .NET should **work well with the .NET Common Language Runtime (CLR).** The CLR provides interoperability with other languages. It also ensures that JScript .NET works well with the .NET Frameworks, a huge library of useful packages.

 

 

4\)      JScript .NET should make **programming in the large easier**. It should support programming styles that make it easy to reason about large programs. It should also provide programming constructs which make it easy to divide up large projects among team members.

 

 

Today I'll talk a bit about the second point -- how can we make JScript .NET faster than JScript?  (I know, I said a while back that I'd give you my rant about performance and script languages.  Maybe if I have time tomorrow.)

 

 

JScript .NET takes a three-pronged approach towards improving performance. Individually each would help somewhat, but as is often the case, the whole is greater than the sum of its parts. These three prongs work together to afford the creation of more performant programs:

 

 

1\)      JScript .NET has a type annotation system.

2\)      JScript .NET discourages and/or restricts certain programming idioms which cause egregious performance problems.

3\)      JScript .NET uses the Common Language Runtime.

 

 

The type annotation system can be complex when you look at the details -- I'll probably talk more about it in detail later.  But essentially the type annotation system is quite simple: the programmer attaches annotations to variable declarations, function argument lists and function return values. These annotations restrict the kinds of data which may be stored, passed or returned respectively. 

 

 

Consider the simple example of adding two and two:

 

 

var Alpha = 2, Beta = 2;

// \[ many lines of code omitted \]

var Gamma = Alpha + Beta;

 

 

Though Alpha and Beta starts off their lives as numbers who knows what they are by the time Gamma is declared? Some code may have stored a string or a date or any other kind of data into Alpha or Beta. 

 

 

Ironically, adding two and two thereby becomes a complicated mess. The JScript engine must determine at run time whether a string has been assigned to either Alpha or Beta (or both). If so then they must be both converted to strings and concatenated. If neither is a string then both must be converted to numbers and added. This sounds simple enough but glosses over a number of details. For instance, the conversions are themselves possibly extremely complicated depending on what data have been stored in Alpha and Beta. Suffice to say that a seemingly simple operation like addition can take thousands of machine cycles, entire microseconds in some cases.

 

 

This is unfortunate. If there is one thing that computers are good at it's adding numbers blindingly fast. If the JScript .NET compiler could **somehow know that only numbers could possibly be stored** in Alpha and Beta then **there would be no need to determine the types at run time**. The compiler could simply emit instructions optimized for the case of adding two numbers. This operation could take nanoseconds instead of microseconds.

 

 

In JScript .NET you annotate the type of a variable by appending a colon and the type after the declaration.

 

 

var Alpha : Number = 2, Beta : Number = 2;

// \[ many lines of code omitted \]

var Gamma : Number = Alpha + Beta;

 

 

The code generated for this addition should now be much more efficient. 

 

 

Type annotation also allows function calls to be much more efficient. Consider a simple function call on a string:

 

 

var Delta = 123;

// \[ many lines of code omitted \]

var Zeta = Delta.toString();

 

 

Again, though Delta starts off as a number it could be anything by the time toString is called. This is what we mean by a late bound scenario. The type of Delta is unknown, so at run time the object must be searched to see if it has a method named "toString". This search is potentially expensive. It could be far more efficient if the compiler knew ahead of time that Delta should be converted to a string the same way that all numbers are converted to strings.

 

 

var Delta : Number = 123;

// \[ many lines of code omitted \]

var Zeta : String = Delta.toString();

 

 

Note that the JScript .NET compiler has an **automatic type inference engine**. **If the compiler sees that a local variable in a function only ever has, say, strings assigned to it then the compiler will treat the variable as though it was annotated with the** String** type.** However, it is a poor programming practice to rely upon the type inference engine. Certain programming practices (especially the use of the eval function) prevent the inferrer from making valid inferences. Also the inferrer can only make inferences on local variables; **the types of unannotated global variables are not inferred.**

 

 

There are also other good reasons to add annotations rather than relying on the inferencer to do it for you, but those will have to wait for another entry.

