# Static analysis of "is"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/12/2012 11:10:00 AM

-----

Before I get into the subject of today's fabulous adventure, I want to congratulate the whole rest of Developer Division on [the tremendously exciting product that we are formally launching today](http://blogs.msdn.com/b/somasegar/archive/2012/09/12/visual-studio-2012-and-net-4-5-launch.aspx). (I've done very little actual coding on Visual Studio 2012 and C\# 5.0, being busy with the long-lead [Roslyn project](http://msdn.microsoft.com/roslyn).) [Asynchronous programming support](http://blogs.msdn.com/b/ericlippert/archive/tags/async/) in C\# and VB is of course my favourite feature; there are far, far too many new features to mention here. Check out the [launch site](http://www.visualstudiolaunch.com), and please do [send us constructive feedback](http://connect.microsoft.com/) on what you do and do not like. We cannot respond in detail to all of it, but it is all appreciated.

-----

Returning now to the subject we started discussing last time on FAIC: sometimes the compiler can know via "static" analysis (that is, analysis done knowing only the compile-time types of expressions, rather than knowing their possibly more specific run-time types) that an "is" operator is guaranteed to produce a particular result.

But before we get into that, let's briefly review the meaning of "is" in C\#. The expression

x is T

for an expression x and a type T produces a bool. Generally speaking, if there is a **reference, boxing or unboxing conversion** from the *runtime* value of x to the type T then the result is true, otherwise the result is false. Note that in particular **user-defined conversions are not considered**. The intention of the operator is to determine if the *runtime* value of x is **actually a useful value** of the type T, (\*) and therefore we add a few additional caveats:

  - T may not be a pointer type
  - x may not be a lambda or anonymous method
  - if x is classified as a method group or is the null literal (\*\*) then the result is false
  - if the runtime type of x is a reference type and its value is a null reference then the result is false
  - if the runtime type of x is a nullable value type and the HasValue property of its value is false then the result is false
  - if the runtime type of x is a nullable value type and the HasValue property of its value is true then the result is computed as though you were checking x.Value is T

So, knowing that, try to think of some situations in which you know for certain that "x is T" is going to always produce true, or always produce false. Here are a few that you and I know are always true:

int i = 123;  
bool b1 = i is int;  
bool b2 = i is IComparable;  
bool b3 = i is object;  
bool b4 = "hello" is string;

Here in every case we know first, that the operand is not null, and second, that the operand will always be of the given type, and therefore the operator will always produce "true".

Before we go on, another aside\! I want to briefly review [our criteria for when to make a warning](http://blogs.msdn.com/b/ericlippert/archive/2011/03/03/danger-will-robinson.aspx): the warning in question has got to be (1) thought of, (2) possible to implement cheaply, (3) identify code that is both plausibly written by a real user and almost certainly wrong, but non-obviously wrong (4) be work-around-able should the code actually be correct and (5) not turn huge amounts of existing code into spurious errors.

Of these four lines, only the third strikes me as plausibly written by a real user and almost certainly wrong; the user must not realize that every int always implements IComparable. The other ones are just weird. Interestingly enough, the C\# 5 compiler warns about the first three, but not the fourth.

There are a lot more situations where you know that the result will always be false. Can you think of some? Here are just a few off the top of my head:

bool b5 = M is Func\<object\>; // M is a method group  
bool b6 = null is object;  
bool b7 = b5 is IEnumerable;  
bool b8 = E.Blah is uint; // E is an enum type  
bool b9 = i is double;

The first two follow the rules of the spec. The latter three are cases where we can know via static analysis that the value cannot possibly be converted to the given type by a reference, boxing or unboxing conversion. We produce a warning for all these trivially-analyzed cases. (Though of course again some of these examples -- b6 in particular -- are unlikely to show up in real code.)

That was a whole lot of preamble to the question I actually want to consider today, which is "how far should we go?" when making these sorts of static analyses for the purpose of warning that an "is" expression is always false. We could go a lot farther\! I started this series off with a puzzle about the cases where there is no compile-time conversion from x to T, but nevertheless "x is T" can be true; today I want to talk about what we do when there is no compile-time conversion from x to T, and hey, x is T really *cannot* possibly be true as a result. There are lots of cases where you and I know that a given expression will never be of a given type, but these cases can get quite complex. Let's look at three complex ones:

class C\<T\> {}  
...  
static bool M10\<X\>(X x) where X : struct { return x is string; }  
static bool M11\<X\>(C\<X\> cx) where X : struct { return cx is C\<string\>; }  
static bool M12\<X\>(Dictionary\<int, int\> d) { return d is List\<X\>; }

In case M10 we know that X will always be a value type and no value type is convertible to string via a reference, boxing or unboxing conversion. The type check must be false.

In case M11 we know that cx is of type C\<some-value-type\>, or a type derived from C\<some-value-type\>. You and I know that there is no way to get the same generic class type into a type hierarchy twice; there is no way to make something derive from both C\<some-value-type\> and C\<string\>. So the type check must be false.

In case M12 we know that there is no way to make an object that inherits from both the dictionary and list base classes, no matter what X is. The type check must be false.

In all these cases we could produce a compiler warning, but we are rapidly running up against the "possible to implement cheaply" criterion\! I could spend a lot of expensive time coming up with heuristics which would not make a difference to real users writing real code. We have to draw the line somewhere.

Where is that line? The heuristic we actually use to determine whether or not to report a warning **when we detect that there is no compile-time conversion from x to T** is as follows:

  - If neither the compile time type of x nor the type T is an open type (that is, a type that involves generic type parameters) then we know that the result will always be false. There's no conversion, and nothing to substitute at runtime to make there be a conversion. Give a warning.
  - One of the types is open. If the compile time type of x is a value type and T is a class type then we know that the result will always be false (\*\*\*). (This is case M10 above.) We give a warning.
  - Otherwise, give up on further analysis and do not produce a warning.

This is far from perfect, but it is definitely in the "good enough" bucket. And "good enough" is, by definition, good enough. Of course, this heuristic is subject to change without notice, should we discover that some real user-impacting scenario motivates changing it.

-----

(\*) And not to determine the answers to other interesting questions, like "is there any way to associate a value of type T with this value?" or "can the value x be legally assigned to a variable of type T?"

(\*\*) There are some small spec holes here and around these holes are minor inconsistencies between the C\# 5 compiler and Roslyn. We do not say what to do if the expression x is a namespace or a type; those are of course valid *expressions*. The compiler produces an error stating that an expression with a value is expected. Similarly for write-only properties and write-only indexers. The C\# 5 compiler produces an error if the expression is a void-returning call, which technically is a spec violation; Roslyn produces a warning, though frankly, I'd be more inclined to change the spec in this case. The specification *does* say that "x is T" is an error if T is a static type; the C\# 5 compiler erroneously allows this and produces false whereas the Roslyn compiler produces an error.

(\*\*\*) **This is a lie**; there is **one** case where the compile time type of x is an open value type and T is a class type, and there is no compile-time conversion from x to T, and "x is T" can be true, and therefore the warning must be suppressed. Can you write a program that demonstrates this scenario?

