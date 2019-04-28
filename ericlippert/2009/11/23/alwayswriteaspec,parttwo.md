# Always write a spec, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/23/2009 7:01:00 AM

-----

Upon submitting that specification for review, even before seeing my code, Chris found a bug and an omission.

The omission is that I neglected to say what happens when the ref variable is an index into a fixed-size array buffer. As it turns out, that case is also rewritten as a pointer dereference by the time we get to this point in the code, so missing that case turned out to not be a big deal.

The bug is that this line is wrong:

-----

if x is ref/out instance.field then add var temp=instance to L and ref/out temp.field to A2.

-----

because it does not meet the stated goal of the method, namely, to produce the same results after the transformation:

 

struct S { public int q; }  
static void M(ref int r) { r = r + 1; }  
static int Zero() { Console.WriteLine("hello\!"); return 0; }  
...  
S\[\] arr = new\[\] { new S() };  
M(ref arr\[Zero()\].q);  
Console.WriteLine(arr\[0\].q); // 1

Here we have a variable expression of the form instance.field which has a side effect; it prints "Hello". So according to the spec, we rewrite that as

 

S temp = arr\[Zero()\];  
M(ref temp.q);  
Console.WriteLine(arr\[0\].q); // 0 \!

Once more, mutable value types are revealed to be pure concentrated evil. We just mutated temp.q, which is a *copy* of arr\[0\].

The interesting thing here is that Chris found the bug by **reading the spec** and thinking about whether I'd missed any interesting cases. Bugs are always cheaper to fix the earlier you find them; a bug you fix as a result of reading the spec is a bug that you don't have to pay QA to find, it's a bug that your customers never see, it's a bug that never causes a backwards-compatibility breaking change, and so on.

I thought hard about this and realized that the painful case only happens when we have ref/out instance.field with a side effect and **instance is a variable of value type**. That's the missing case. Furthermore, we can solve this by recursing\!

-----

  - if x is ref/out instance.field and instance is a variable of value type, then recurse. Compute T("ref instance"), as if we were invoking a method that takes a single parameter of this ref type. That will give us a resulting declaration list, which we merge onto the end of L, and a resulting argument list with one element on it, r. Add "ref r.field" to A2.
  - if x is a ref/out instance.field but instance is not a variable of value type, then generate var temp=instance on L,  ref temp.field on A2.

-----

So if we have, say, M(ref arr\[index\].q), that ultimately is generated as var t1 = arr, var t2 = index, M(ref t1\[t2\].q), which is correct.

I fixed up my implementation, wrote some test cases to test the spec, and sent it off to testing for further analysis.

There's *still* a bug in this spec, which our tester David found during his analysis of my spec plus proposed implementation. Hint: the problem has nothing to do with ref/out semantics; it's a more fundamental error in reasoning.

Feel free to pause and reflect for a moment if you want a chance to figure it out for yourself.

...

The error is in the default case:

-----

First, if x has no side effects, then simply keep x in the appropriate place in A2.

-----

Whether x has side effects or not is completely irrelevant; what is relevant is whether the computation of x depends on the order of every other side effect\!

For example, our spec would have that

 

int k = 0;  
M( k, arr\[k=1\] );

is the same as

 

int k = 0;  
var t1 = arr;  
var t2 = (k = 1);  
M(k, t1\[t2\]);

which is clearly not correct; the first argument is supposed to be zero, not one. The fact that the first argument has no side effects is irrelevant; it depends on the second argument's side effect taking effect *after* the first argument's *value* is computed.

We have to fix up the spec (and implementation) to say that if the argument corresponds to a *value* parameter, you *always* save away the value in a temporary, regardless of whether it has side effects or not. If it corresponds to, say, a ref parameter, and the argument is a ref to a local then we can still simply generate the ref to the local normally; regardless of whether the contents of the local change, the managed address of the local is always the same, and computing the address of a local causes no side effects.

I note that we could also make an exception in the case that the value is, say, a compile-time constant. Clearly the computation of its value does not have any dependency on the ordering of other side effects. But for the sake of not further complexifying my spec and implementation, I glossed over this detail. The optimizer will probably take care of it.

Commenter Pavel Minaev found several more corner cases that none of us found. The big one is that we should be clear about what happens with multidimensional arrays, and in fact, I had a bug in my implementation for multidimensional arrays, so thanks Pavel\! He also found some obscure ones. One is that accessing a static field does have a side effect; if this is the first access to the class then that causes the static class constructor to run, which could have observable side effects. Another is that the exact semantics of when bad arguments cause exceptions is slightly changed by this transformation.

At this point I now believe that I have a correct specification of my function T (modulo Pavel's weird issues, which I'm still mulling over. But those are bizarre corner cases that might require some language spec work to sort out.) Since the implementation is a straightforward transformation of the specification into C++, I also believe I have a correct implementation. I've written test cases that verify every case in the spec, and handed it off to testing to look for practical cases I missed.

Anyone see any other problems with this spec? Anyone?

And anyone have any guesses as to why I needed this function implemented?

