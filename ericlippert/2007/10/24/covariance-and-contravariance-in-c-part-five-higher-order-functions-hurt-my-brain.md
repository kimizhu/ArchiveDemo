<div id="page">

# Covariance and Contravariance In C\#, Part Five: Higher Order Functions Hurt My Brain

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/24/2007 10:06:00 AM

-----

<div id="content">

<div class="mine">

Last time I discussed how we could in a hypothetical future version of C\# allow delegate types to be covariant in their return type and contravariant in their formal parameter types. For example, we could have a contravariant action delegate:

<span class="code"> </span>

delegate void Action\< -A \> (A a);

and then have

<span class="code"> </span>

Action\<Animal\> action1 = (Animal a)=\>{ Console.WriteLine(a.LatinName); };  
Action\<Giraffe\> action2 = action1;

Because <span class="code">action2</span>’s caller will always pass in something that <span class="code">action1</span> can handle.

Based on my discussion so far, I hope that you have a strong intuition that the normal, sensible use of variance is *“stuff going ‘in’ may be contravariant, stuff going ‘out’ may be covariant”.* Though I believe that would be the most common use of variance were we to enable this feature in a hypothetical future version of C\#, the real situation would actually be rather more complicated than that. There *is* a situation where it is legal to use a *covariant* parameter in the formal parameter of a delegate. Doing so makes my brain hurt, but this also builds character, so here we go\!

Suppose you want to do "higher order" functional programming. For example, perhaps you want to define a meta-action – a delegate which takes actions and does something with them:

<span class="code"> </span>

delegate void Meta\<A\> (Action\<A\> action);

for example,

<span class="code"> </span>

Meta\<Mammal\> meta1 = (Action\<Mammal\> action)=\>{action(new Giraffe());};  
// The next line is legal because Action\<Animal\> is smaller than Action\<Mammal\>;  
// remember, Action is contravariant  
meta1(action1);

So this <span class="code">Meta</span> thing takes an <span class="code">Action</span> on <span class="code">Mammal</span>s – say, <span class="code">action1</span> above, which prints the Latin name of any <span class="code">Animal</span>, and therefore can do so to any <span class="code">Mammal</span> – and then invokes *that* action on a new <span class="code">Giraffe</span>.

Clearly the type parameter <span class="code">A</span> is used in an *input* position in the definition of <span class="code">Meta\<A\></span>, so we should be able to make it *contravariant*, right? Suppose we did so. That would mean that this assignment would be legal:

<span class="code"> </span>

Meta\<Tiger\> meta2 = meta1; // would be legal if Meta were contravariant in its parameter

But that means that this would then be legal:

<span class="code"> </span>

Action\<Tiger\> action3 = tiger=\>{ tiger.Growl(); };  
meta2(action3);

Follow the logic through and you’ll see that we end up calling <span class="code">(new Giraffe()).Growl()</span>, which is clearly a violation of both the type system and of nature’s laws.

Thus <span class="code">Meta\<A\></span> *cannot* be *contravariant* in <span class="code">A.</span> It can however be *covariant*:

<span class="code"> </span>

Meta\<Animal\> meta3 = meta1; // legal if Meta were covariant

Now everything works. <span class="code">meta3</span> takes an <span class="code">Action</span> on <span class="code">Animal</span>s and passes a <span class="code">Giraffe</span> to that <span class="code">Action</span>. We’re all good.

Contravariance is tricky; the fact that it *reverses* the bigger/smaller relationship between types means that a type parameter used in a "doubly contravariant" position (being an input of <span class="code">Action</span>, which is itself an input of <span class="code">Meta</span>) becomes *covariant*. The second reversal undoes the first.

**We do not expect that most people will use variance in this manner**; rather, we expect that people will almost always use covariant parameters in output positions and contravariant parameters in input positions. As we’ll see a bit later in this series, whether this expectation is reasonable or not would influence the syntax we might choose were we to add variance to a hypothetical future version of C\#.

Next time: we’ll leave delegates behind and talk about variance in interfaces.

</div>

</div>

</div>

