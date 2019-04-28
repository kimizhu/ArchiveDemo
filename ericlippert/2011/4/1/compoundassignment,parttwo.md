# Compound Assignment, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2011 1:06:00 AM

-----

[Last time](http://blogs.msdn.com/b/ericlippert/archive/2011/03/29/compound-assignment-part-one.aspx) I discussed how the compound assignment operators of the form “x op= y” have some perhaps unobvious behaviours in C\#, namely:

(1) though logically this is expanded as “x = x op y”, x is only evaluated once  
(2) for built-in operators, if necessary, a cast is inserted, so that this is analyzed as “x = (T)(x op y)”  
(3) for built-in operators, if “x = y” would be illegal then so is “x op= y”

I am pleased to announce that we are at long last filling in some holes in the C\# language. We at present support compound assignment on the addition, subtraction, division, remainder, multiplication, left shift, right shift, bitwise and, bitwise or and bitwise xor operators, but that leaves out a large number of operators. In the next version of C\# we will be adding compound assignment operators for most of the remaining binary (and ternary\!) operators.

Doing so requires us to relax some of the rules above; I'll describe why below.

Let's start with an easy one.

**Equality/Inequality**

In the next version of C\#, x \!== y will mean x = (x \!= y).

Now, for all the built-in types, x \!= y returns bool, so on the built-in types this syntax only works if x and y are both bool. This was actually the easiest of the new compound operators to implement because it already was implemented\! x \!== y on bools is the same as x ^= y, which we already had.

Similarly, x === y is the opposite; it is the same as x = \!(x^y). Or is it the same as x = (\!x)^y? Turns out, both\! Is that ever weird\!

I am slightly worried that people conversant with JScript will confuse the meaning of === in JScript (equality without type conversion) and === in C\# (compound assignment with equality) but I hope it will not be too confusing.

The compound equality and inequality operators are trivial; what about the other comparison operators? Can we make them into compound operators?

**Comparison**

Were there any justice in this world, x \<= y ought to mean x = x \< y, but unfortunately \<= is already taken. Similarly with x \>= y. Regrettably we do not have a compound assignment syntax for these, but it's probably just as well, as you'll see in the next point:

x \>== y and x \<== y mean x = x \>= y and x = x \<= y, respectively. Now, for all the built-in types upon which they are defined, \<= and \>= return bool, but there are no \<= or \>= operators defined on bool (or object) and therefore, the compound operators are useful on no built-in types. However, we will support them. If you create a user-defined type, call it C, such that C has a user-defined explicit or implicit conversion from bool, and a user-defined \<= operator, then you can say x \<== y for expressions x and y of type C. It expands as x = (x \<= y), of course, modulo that x is computed only once.

**Conditional**

One more before we leave the "bool" trap. An exciting opportunity here is to extend the compound assignment operators from binary operators to ternary operators. Now, C\# only has one ternary operator, the conditional operator, but in general we can treat any "infix" ternary operator as two binary operators. That is, rather than thinking of x ? y : z as a ternary operator, think of it as two binary operators, ? and :, which always appear together. Once you think of it like that then you can see how we can make this into a compound assignment operator:

x ?= y : z means x = x ? y : z

Clearly they must all be bools or types convertible to bool. In addition, we strengthen the requirement of the third rule: for built-in types both y and z must be assignable to the type of x.

A frequent question on StackOverflow is why the conditional operator does not take into account the type to which it is being converted; in this syntax, it does because again, a cast is inserted if necessary if the types are built-in types. Hopefully that will decrease user confusion at least slightly.

**Coalescing**

x ??= y means x = x ?? y

This one is actually pretty straightforward and very useful. The type analysis of ?? is a bit odd (see the spec for details) but the semantics of the operator already require that the left and right sides have type compatibility, at least modulo nullability. The operator basically means "if the left side is null, replace it with the right side, otherwise keep it the same and use its value".

Thus far we've seen the easy ones. In the remaining operators we completely remove the restriction that "y" be assignable to "x"; the reasons will become clear.

**Type comparison**

x is= Y means x = x is Y

again, since "is" only returns bool, the only types this works on are bool and object. For example:

object x = "hello";  
x is= Exception;

means x = (x is Exception), so x becomes a boxed "false". Similarly:

x as= Y means x = x as Y

This one essentially keeps x the same if it is of type Y, and turns it to null if it is not.

We relax the restriction that x = Y be legal for built-in operators in both these cases, since it practically never will be.

**Call, index and member access**

x ()= y means x = x(y)

This one takes some thinking. Clearly x must be of delegate type so that it can be invoked. And the delegate invoked must return a delegate assignable to x. [I did an article on delegates like that a while back](http://blogs.msdn.com/b/ericlippert/archive/2006/06/23/standard-generic-delegate-types-part-two.aspx); essentially this operator works best on combinators:

delegate D D(D d); // D is a delegate which takes a D and returns a D.  
void M()  
{  
    D x = q=\>q;  
    D y = r=\>s=\>r(s);  
    x ()= y;  
    // means x = x(y), which in this particular case, just assigns y to x.  
}

Clearly in this example it is very useful that y be assignable to x, but it is not required.

Now, you might say, doesn't this already have a meaning? That is, x() = y means assign the value of y to the variable x(). But in C\#, the result of a method invocation is never a variable, it is always a value (or void).

This is not true in general in the CLR; as I noted last time, **the type system supports methods that return an alias to a variable**. If we ever want to add variable-returning methods to C\#, this is going to be tricky. We considered not adding this compound assignment operator because it might make it more difficult to add the variable-returning-method feature in the future. But a great feature today is worth it; we might never implement the variable-returning-method feature and it seems a shame to deprive people of this awesome syntax as a result of a hypothetical future feature.

Similarly to invocation, we can do indexing. x \[\]= y means x = x\[y\]. The typical case is that x is a type that contains an indexer which returns an instance of that type.

A slightly odd one that was controversial in the design meetings was member access. x .= Y means x = x.Y -- clearly Y must be a field or property of a type compatible with x, **or a method group such that x is a delegate type compatible with it**. (That would restrict Y to be names of methods of a delegate type, like Invoke.)

**Stuff we're not supporting**

That leaves the new, anonymous method and lambda operators. After much debate we decided to not support x new= Y, x delegate={y} or my personal favourite, x=\>=y. Note that the latter would mean x = x=\>y, which violates [the rule that the same simple name not have two meanings in the same block](http://blogs.msdn.com/b/ericlippert/archive/2009/11/02/simple-names-are-not-so-simple.aspx).

We hope you enjoy these new operators; I'm hoping we can do a second preview release of the compiler soon so that you can experiment with how these operators interact with async/await\!

.

.

.

.

 

**UPDATE: HA HA HA HA HA HA HA HA HA HA HA HA HA HA HA\! I totally crack myself up.**

In case it is not clear -- and based on the number of comments I got of the form "*Are you serious or is this an April Fools Day joke?*" it was exactly the right amount of unclear -- this is a joke; we are not adding any of these operators. [Part One is of course perfectly serious](http://blogs.msdn.com/b/ericlippert/archive/2011/03/29/compound-assignment-part-one.aspx). 

One of the most common comments is that "??=" actually is useful. Indeed, it would be pretty useful, though we have no plans to do it at this time. It was a comment suggesting ??= on [last year's April Fool's Day post](http://blogs.msdn.com/b/ericlippert/archive/2010/04/01/somelastminutefeatures.aspx) that inspired this one.

