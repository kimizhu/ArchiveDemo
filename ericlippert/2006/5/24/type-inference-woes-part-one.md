<div id="page">

# Type inference woes, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/24/2006 3:16:00 PM

-----

<div id="content">

<div class="mine">

The C\# compiler has a subtle violation of the specification which raises an interesting question for some of the new LINQ-related features. The specification for the <span class="code">?:</span> operator states the following:

<div class="spec">

The second and third operands of the ?: operator control the type of the conditional expression. Let X and Y be the types of the second and third operands. Then,

  - If X and Y are the same type, then this is the type of the conditional expression.
  - Otherwise, if an implicit conversion exists from X to Y, but not from Y to X, then Y is the type of the conditional expression.
  - Otherwise, if an implicit conversion exists from Y to X, but not from X to Y, then X is the type of the conditional expression.
  - Otherwise, no expression type can be determined, and a compile-time error occurs.

</div>

This makes a lot of sense. If you have

<div class="code">

Giraffe g = (a \> b) ? new Giraffe() : new Mammal();

</div>

then you ought to get a type error. The type of the ternary expression should be <span class="code">Mammal</span>, not <span class="code">Giraffe</span>, because <span class="code">Giraffe</span> goes to <span class="code">Mammal</span> but <span class="code">Mammal</span> does not go to <span class="code">Giraffe</span>.

If both types go to the other (which can happen if both types define an implicit conversion operator for the other) then we can't decide which one is better and give up.

If neither type goes to the other, then notice that we do not attempt to find the "nearest encompassing type". For example, if we had

<div class="code">

Mammal m = (a \> b) ? new Dog() : new Cat();

</div>

then we would again get a type error. We want to be able to have a unique type as the type of the ternary expression. We do not want to get into the business of saying "well, <span class="code">Dog</span> and <span class="code">Cat</span> are both subclasses of <span class="code">Mammal</span>, but they are also both implementors of <span class="code">IHousepet</span>, so which one is the "real" nearest encompassing type?" It's just too big a can of worms. We like the principle that the type of the expression must be the type of something in the expression.

So where's the spec violation? Well, according to the specification above, is this legal?

<div class="code">

short s = 123;  
int i = (a \> b) ? 0 : s;

</div>

X is <span class="code">int</span>, Y is <span class="code">short</span>. Y goes to X, X does not go to Y, therefore X is the type of the expression, so this should work. But in fact, C\# reports this as an error, because the rule C\# actually implements is:

<div class="spec">

Let B and C be the second and third operands. Let X and Y be the types of the second and third operands. Then,

  - If X and Y are the same type, then this is the type of the conditional expression.
  - Otherwise, if an implicit conversion exists from B to Y, but not from C to X, then Y is the type of the conditional expression.
  - Otherwise, if an implicit conversion exists from C to X, but not from B to Y, then X is the type of the conditional expression.
  - Otherwise, no expression type can be determined, and a compile-time error occurs.

</div>

Since the types are not the same, and literal zero goes to <span class="code">short</span>, and <span class="code">s</span> goes to <span class="code">int</span>, no expression type can be determined and we report an error.

One could make the argument that this is the better behaviour. It certainly does seem that in this case it's arguably ambiguous what the best type for the expression is since both halves can be converted to both <span class="code">int</span> and <span class="code">short</span>.

Now, since this is an error case, surely we can simply fix the bug. We'd be turning a case which is presently an error into something which works. Turning broken code into working code is by definition not a breaking change -- a breaking change is turning working code into broken code, or working code with different semantics.

Unfortunately, fixing this would be a breaking change:

<div class="code">

public delegate int D();  
...  
D d = (a \> b) ? (D)(delegate() { return 1; }) : delegate() { return 2; };

</div>

Notice that the latter expression has no type at all. Yes, anonymous methods expressions are expressions without a type. If we used the "expression" version of the algorithm then the second expression is convertible to the type of the first, but the first is not convertible to the non-existant type of the second, so the type of the expression is the type of the first. If we used the spec-compliant "type" version then the second expression doesn't even have a type, so there's no type we can infer\!

Next time I'll talk about an algorithm that might save the day here, and also discuss how we'll need to generalize this to make some of the LINQ-related language features work.

</div>

</div>

</div>

