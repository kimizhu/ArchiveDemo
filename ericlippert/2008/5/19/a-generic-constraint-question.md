<div id="page">

# A Generic Constraint Question

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/19/2008 10:01:00 AM

-----

<div id="content">

<div class="mine">

Here's a question I get fairly frequently: the user desires to create a generic type <span class="code">Bar\<T\></span> constrained such that <span class="code">T</span> is guaranteed to be a <span class="code">Foo\<U\></span>, for some <span class="code">U</span>.

The way they usually try to write this is

<span class="code"> </span>

class Bar\<T\>  where T : Foo\<T\> {...

which of course then does not work. The user discovers this when they type

<span class="code"> </span>

 new Bar\<Foo\<string\>\>()

and get an error stating that the constraint has been violated. Which it certainly has. The constraint says that <span class="code">T</span> must be <span class="code">Foo\<T\></span>, so therefore <span class="code">string</span> must be <span class="code">Foo\<string\></span>, which clearly it is not.

There are a few ways to solve this problem. One is to simply introduce a new type parameter:

<span class="code"> </span>

 class Bar\<T, U\> where T : Foo\<U\>

this means that you now have some redundancy:

<span class="code"> </span>

new Bar\<Foo\<string\>, string\>()

But let's stop and think for a moment about what the user desired in the first place. Suppose there were a way to specify what we sometimes call a "mumble type" on the C\# design team:

<span class="code"> </span>

class Bar\<T\>  where T : Foo\<?\> {...

Think about the consequences of that on the caller side; the compiler could verify that <span class="code">new Bar\<Foo\<string\>\>()</span> was legal. But what could the compiler verify on the callee side?

<span class="code"> </span>

class Bar\<T\>  where T : Foo\<?\> {  
   public void Blah(T t) {  
    t.

t dot what? We don't know anything about <span class="code">T</span> except that it is <span class="code">Foo\<something\></span>. What methods, properties, fields or events of <span class="code">Foo</span> could we access? Only those that in no way consume the generic type, which seems like precious little. If <span class="code">Foo</span> had a whole lot of stuff that wasn't generic, why was it made generic in the first place?

But this then leads to an insight; if the user owns the <span class="code">Foo\<T\></span> class, then they could do precisely that:

<span class="code"> </span>

public abstract class FooBase  
{  
  private FooBase() {} // Not inheritable by anyone else  
  public class Foo\<U\> : FooBase {...generic stuff ...}  
  ... nongeneric stuff ...  
}

public class Bar\<T\> where T: FooBase { ... }  
...  
new Bar\<FooBase.Foo\<string\>\>()

and now everyone is happy. The compiler restricts the construction of <span class="code">Bar</span> to <span class="code">FooBase</span> on the caller side. The compiler ensures that every runtime instance of <span class="code">FooBase</span> is an instance of <span class="code">Foo\<T\></span>. And the compiler allows the callee side to use all the non-generic methods on <span class="code">FooBase</span>.

Personally, I would go for the first solution myself. But it's edifying to try and find additional solutions, even if they are a bit odd.

</div>

</div>

</div>

