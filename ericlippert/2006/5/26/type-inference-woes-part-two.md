<div id="page">

# Type inference woes, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/26/2006 12:29:00 PM

-----

<div id="content">

So what's the big deal anyway? The difference between the spec and the implementation is subtle, only affects a few specific and rather unlikely scenarios, and can always be worked around by inserting casts if you need to. Fixing the implementation would be a breaking change, it seems like a small and simple change to the specification, so why don't we just update the specification the next time we get the chance and be done with it?

The big deal is that this is a small, isolated, corner case problem for C\# 2.0, but it becomes much more visible in C\# 3.0. Essentially the question here is "given a set of expressions of various types, how do we infer a unique unified type?" In C\# 2.0 this question comes up only in the context of the <span class="code">?:</span> operator, and the set always has two elements. In C\# 3.0, this question comes up all over the place and the sets can be arbitrarily large. For example:

<div class="mine">

  - When an implicitly typed local variable declaration statement contains several declarations, does  
      
    <span class="code">const short s = 123;  
    var x = 0, y = s;  
      
    </span>infer that <span class="code">x</span> and <span class="code">y</span> are <span class="code">short</span>, or <span class="code">int</span>, or is this an error?
  - Does the implicitly typed array initializers  
      
    <span class="code">const short s = 123;  
    var x = new\[\] {0, s};  
      
    </span>infer <span class="code">x</span> to be <span class="code">short\[\]</span>, <span class="code">int\[\]</span>, or is this an error?
  - When a lambda expression is passed to a generic method we must infer the return type of the lambda from the set of expressions returned from its body:  
      
    <span class="code">public static IEnumerable\<T\> Select\<S, T\>(IEnumerable\<S\> collection, Func\<S, T\> projection){//...  
    ...  
    const short s = 123;  
    var x = Select(blahCollection, c =\> { if (c.Foo \> c.Bar) return 0; else return s; });  
      
    </span>If <span class="code">S</span> is inferred to be <span class="code">Blah</span>, is <span class="code">x</span> inferred to be <span class="code">IEnumerable\<short\></span>, or <span class="code">IEnumerable\<int\></span>, or is this an error?
  - When multiple lambda expressions are passed to a generic method, can we unify unequal inferred types?  
      
    <span class="code">public static IEnumerable\<T\> Join\<O, I, K, R\>(  
    IEnumerable\<O\> outer,  
    IEnumerable\<I\> inner,  
    Func\<O, K\> outerkey,  
    Func\<I, K\> innerkey,  
    Func\<O, I, R\> selector) { // ...  
      
    var x = Join(customers, orders, c =\> c.Id, o =\> o.CustId,  
    (c, o) =\> new {c.Name, o.Amount});  
      
    </span>

</div>

<div class="mine">

Suppose <span class="code">Customer.Id</span> is <span class="code">int</span> and <span class="code">Order.CustId</span> is <span class="code">Nullable\<int\></span>. Do we infer that <span class="code">K</span> is <span class="code">Nullable\<int\></span>, or produce an error? Enquiring minds want to know the answers to these questions, and it seems sensible that we should come up with a single algorithm that answers all of them. And if we're going to do that, then it seems desirable that <span class="code">?:</span> ought to use the same algorithm we come up with for all of the above.

After the Memorial Day break I'll discuss some of the algorithms that we're considering, and what benefits and drawbacks they have. Have a good weekend\!

</div>

</div>

</div>

