<div id="page">

# Compound Assignment, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2011 1:06:00 AM

-----

<div id="content">

<div class="mine">

[Last time](http://blogs.msdn.com/b/ericlippert/archive/2011/03/29/compound-assignment-part-one.aspx) I discussed how the compound assignment operators of the form “<span class="code">x op= y</span>” have some perhaps unobvious behaviours in C\#, namely:

(1) though logically this is expanded as “<span class="code">x = x op y</span>”, x is only evaluated once  
(2) for built-in operators, if necessary, a cast is inserted, so that this is analyzed as “<span class="code">x = (T)(x op y)</span>”  
(3) for built-in operators, if “<span class="code">x = y</span>” would be illegal then so is “<span class="code">x op= y</span>”

I am pleased to announce that we are at long last filling in some holes in the C\# language. We at present support compound assignment on the addition, subtraction, division, remainder, multiplication, left shift, right shift, bitwise and, bitwise or and bitwise xor operators, but that leaves out a large number of operators. In the next version of C\# we will be adding compound assignment operators for most of the remaining binary (and ternary\!) operators.

Doing so requires us to relax some of the rules above; I'll describe why below.

Let's start with an easy one.

**Equality/Inequality**

In the next version of C\#, <span class="code">x \!== y</span> will mean <span class="code">x = (x \!= y)</span>.

Now, for all the built-in types, <span class="code">x \!= y</span> returns bool, so on the built-in types this syntax only works if x and y are both bool. This was actually the easiest of the new compound operators to implement because it already was implemented\! <span class="code">x \!== y</span> on bools is the same as <span class="code">x ^= y</span>, which we already had.

Similarly, <span class="code">x === y</span> is the opposite; it is the same as <span class="code">x = \!(x^y)</span>. Or is it the same as <span class="code">x = (\!x)^y</span>? Turns out, both\! Is that ever weird\!

I am slightly worried that people conversant with JScript will confuse the meaning of === in JScript (equality without type conversion) and === in C\# (compound assignment with equality) but I hope it will not be too confusing.

The compound equality and inequality operators are trivial; what about the other comparison operators? Can we make them into compound operators?

**Comparison**

Were there any justice in this world, <span class="code">x \<= y</span> ought to mean <span class="code">x = x \< y</span>, but unfortunately <span class="code">\<=</span> is already taken. Similarly with <span class="code">x \>= y</span>. Regrettably we do not have a compound assignment syntax for these, but it's probably just as well, as you'll see in the next point:

<span class="code">x \>== y</span> and <span class="code">x \<== y</span> mean <span class="code">x = x \>= y</span> and <span class="code">x = x \<= y</span>, respectively. Now, for all the built-in types upon which they are defined, <span class="code">\<=</span> and <span class="code">\>=</span> return bool, but there are no <span class="code">\<=</span> or <span class="code">\>=</span> operators defined on bool (or object) and therefore, the compound operators are useful on no built-in types. However, we will support them. If you create a user-defined type, call it C, such that C has a user-defined explicit or implicit conversion from bool, and a user-defined <span class="code">\<=</span> operator, then you can say <span class="code">x \<== y</span> for expressions x and y of type C. It expands as <span class="code">x = (x \<= y)</span>, of course, modulo that x is computed only once.

**Conditional**

One more before we leave the "bool" trap. An exciting opportunity here is to extend the compound assignment operators from binary operators to ternary operators. Now, C\# only has one ternary operator, the conditional operator, but in general we can treat any "infix" ternary operator as two binary operators. That is, rather than thinking of <span class="code">x ? y : z</span> as a ternary operator, think of it as two binary operators, ? and :, which always appear together. Once you think of it like that then you can see how we can make this into a compound assignment operator:

<span class="code">x ?= y : z</span> means <span class="code">x = x ? y : z</span>

Clearly they must all be bools or types convertible to bool. In addition, we strengthen the requirement of the third rule: for built-in types both y and z must be assignable to the type of x.

A frequent question on StackOverflow is why the conditional operator does not take into account the type to which it is being converted; in this syntax, it does because again, a cast is inserted if necessary if the types are built-in types. Hopefully that will decrease user confusion at least slightly.

**Coalescing**

<span class="code">x ??= y</span> means <span class="code">x = x ?? y</span>

This one is actually pretty straightforward and very useful. The type analysis of ?? is a bit odd (see the spec for details) but the semantics of the operator already require that the left and right sides have type compatibility, at least modulo nullability. The operator basically means "if the left side is null, replace it with the right side, otherwise keep it the same and use its value".

Thus far we've seen the easy ones. In the remaining operators we completely remove the restriction that "y" be assignable to "x"; the reasons will become clear.

**Type comparison**

<span class="code">x is= Y</span> means <span class="code">x = x is Y</span>

again, since "is" only returns bool, the only types this works on are bool and object. For example:

<span class="code">object x = "hello";  
x is= Exception;</span>

means <span class="code">x = (x is Exception)</span>, so x becomes a boxed "false". Similarly:

<span class="code">x as= Y</span> means <span class="code">x = x as Y</span>

This one essentially keeps x the same if it is of type Y, and turns it to null if it is not.

We relax the restriction that <span class="code">x = Y</span> be legal for built-in operators in both these cases, since it practically never will be.

**Call, index and member access**

<span class="code">x ()= y</span> means <span class="code">x = x(y)</span>

This one takes some thinking. Clearly x must be of delegate type so that it can be invoked. And the delegate invoked must return a delegate assignable to x. [I did an article on delegates like that a while back](http://blogs.msdn.com/b/ericlippert/archive/2006/06/23/standard-generic-delegate-types-part-two.aspx); essentially this operator works best on combinators:

<span class="code">delegate D D(D d); // D is a delegate which takes a D and returns a D.  
void M()  
{  
    D x = q=\>q;  
    D y = r=\>s=\>r(s);  
    x ()= y;  
    // means x = x(y), which in this particular case, just assigns y to x.  
}</span>

Clearly in this example it is very useful that y be assignable to x, but it is not required.

Now, you might say, doesn't this already have a meaning? That is, <span class="code">x() = y</span> means assign the value of y to the variable x(). But in C\#, the result of a method invocation is never a variable, it is always a value (or void).

This is not true in general in the CLR; as I noted last time, **the type system supports methods that return an alias to a variable**. If we ever want to add variable-returning methods to C\#, this is going to be tricky. We considered not adding this compound assignment operator because it might make it more difficult to add the variable-returning-method feature in the future. But a great feature today is worth it; we might never implement the variable-returning-method feature and it seems a shame to deprive people of this awesome syntax as a result of a hypothetical future feature.

Similarly to invocation, we can do indexing. <span class="code">x \[\]= y</span> means <span class="code">x = x\[y\]</span>. The typical case is that x is a type that contains an indexer which returns an instance of that type.

A slightly odd one that was controversial in the design meetings was member access. <span class="code">x .= Y</span> means <span class="code">x = x.Y</span> -- clearly Y must be a field or property of a type compatible with x, **or a method group such that x is a delegate type compatible with it**. (That would restrict Y to be names of methods of a delegate type, like Invoke.)

**Stuff we're not supporting**

That leaves the new, anonymous method and lambda operators. After much debate we decided to not support <span class="code">x new= Y</span>, <span class="code">x delegate={y}</span> or my personal favourite, <span class="code">x=\>=y</span>. Note that the latter would mean <span class="code">x = x=\>y</span>, which violates [the rule that the same simple name not have two meanings in the same block](http://blogs.msdn.com/b/ericlippert/archive/2009/11/02/simple-names-are-not-so-simple.aspx).

We hope you enjoy these new operators; I'm hoping we can do a second preview release of the compiler soon so that you can experiment with how these operators interact with async/await\!

.

.

.

.

 

**UPDATE: HA HA HA HA HA HA HA HA HA HA HA HA HA HA HA\! I totally crack myself up.**

In case it is not clear -- and based on the number of comments I got of the form "*Are you serious or is this an April Fools Day joke?*" it was exactly the right amount of unclear -- this is a joke; we are not adding any of these operators. [Part One is of course perfectly serious](http://blogs.msdn.com/b/ericlippert/archive/2011/03/29/compound-assignment-part-one.aspx). 

One of the most common comments is that "??=" actually is useful. Indeed, it would be pretty useful, though we have no plans to do it at this time. It was a comment suggesting ??= on [last year's April Fool's Day post](http://blogs.msdn.com/b/ericlippert/archive/2010/04/01/somelastminutefeatures.aspx) that inspired this one.

</div>

</div>

</div>

