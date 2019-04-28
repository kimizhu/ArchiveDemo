<div id="page">

# Covariance and Contravariance in C\#, Part Three: Method Group Conversion Variance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/19/2007 10:13:00 AM

-----

<div id="content">

<div class="mine">

Last time I discussed how array covariance is broken in C\# (and Java, and a number of other languages as well.) Today, a non-broken kind of variance supported by C\# 2.0: conversions from method groups to delegates. This is a more complicated kind of variance, so let me spell it out in more detail.

Suppose that you have a method which returns a <span class="code">Giraffe</span>:

<span class="code"> </span>

static Giraffe MakeGiraffe() { …

Suppose further that you have a delegate type representing a function which takes no arguments and returns an <span class="code">Animal</span>. Say, <span class="code">Func\<Animal\></span>. Should this implicit conversion from method group to delegate be legal?

<span class="code"> </span>

Func\<Animal\> func = MakeGiraffe;

The caller of func is expecting an <span class="code">Animal</span> to be returned. The actual function captured by the delegate always returns a <span class="code">Giraffe</span>, which is an <span class="code">Animal</span>, so the caller of <span class="code">func</span> is never going to get anything that they’re not capable of dealing with. There is no problem in the type system here. Therefore *we can make method group to delegate conversions **covariant*** (‡) ***in their return types**.*

Now suppose you have two methods, one which takes a <span class="code">Giraffe</span> and one which takes an <span class="code">Animal</span>:

<span class="code"> </span>

void Foo(Giraffe g) {}  
void Bar(Animal a) {}

and a delegate to a <span class="code">void</span>-returning function that takes a <span class="code">Mammal</span>:

<span class="code"> </span>

Action\<Mammal\> action1 = Foo; // illegal  
Action\<Mammal\> action2 = Bar; // legal

Why is the first assignment illegal? Because the caller of <span class="code">action1</span> can pass a <span class="code">Tiger</span>, but <span class="code">Foo</span> cannot take a <span class="code">Tiger</span>, only a <span class="code">Giraffe</span>\! The second assignment is legal because <span class="code">Bar</span> can take any <span class="code">Animal</span>.

In our previous example we preserved the direction of the assignability: <span class="code">Giraffe</span> is smaller than <span class="code">Animal</span>, so a method which returns a <span class="code">Giraffe</span> is smaller than a delegate which returns an <span class="code">Animal</span>. In this example we are reversing the direction of the assignability: <span class="code">Mammal</span> is smaller than <span class="code">Animal</span>, so a method which takes an <span class="code">Animal</span> is smaller than a delegate which takes a <span class="code">Mammal</span>. Because the direction is reversed, *method group to delegate conversions are **contravariant** **in their argument types***.

Note that all of the above applies only to *reference* types. We never say something like “well, every <span class="code">int</span> fits into a <span class="code">long</span>, so a method which returns an <span class="code">int</span> is assignable to a variable of type <span class="code">Func\<long\></span>”.

Next time: a stronger kind of delegate variance that we could support in a hypothetical future version of C\#.

(‡) A note to nitpickers out there: yes, I said earlier that variance was a property of *operations on types*, and here I have an operation on *method groups*, which are *typeless* expressions in C\#. I’m writing a blog, not a dissertation; deal with it\!  

</div>

</div>

</div>

