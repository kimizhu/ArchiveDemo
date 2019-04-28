# Covariance and Contravariance In C\#, Part Five: Higher Order Functions Hurt My Brain

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/24/2007 10:06:00 AM

-----

Last time I discussed how we could in a hypothetical future version of C\# allow delegate types to be covariant in their return type and contravariant in their formal parameter types. For example, we could have a contravariant action delegate:

 

delegate void Action\< -A \> (A a);

and then have

 

Action\<Animal\> action1 = (Animal a)=\>{ Console.WriteLine(a.LatinName); };  
Action\<Giraffe\> action2 = action1;

Because action2’s caller will always pass in something that action1 can handle.

Based on my discussion so far, I hope that you have a strong intuition that the normal, sensible use of variance is *“stuff going ‘in’ may be contravariant, stuff going ‘out’ may be covariant”.* Though I believe that would be the most common use of variance were we to enable this feature in a hypothetical future version of C\#, the real situation would actually be rather more complicated than that. There *is* a situation where it is legal to use a *covariant* parameter in the formal parameter of a delegate. Doing so makes my brain hurt, but this also builds character, so here we go\!

Suppose you want to do "higher order" functional programming. For example, perhaps you want to define a meta-action – a delegate which takes actions and does something with them:

 

delegate void Meta\<A\> (Action\<A\> action);

for example,

 

Meta\<Mammal\> meta1 = (Action\<Mammal\> action)=\>{action(new Giraffe());};  
// The next line is legal because Action\<Animal\> is smaller than Action\<Mammal\>;  
// remember, Action is contravariant  
meta1(action1);

So this Meta thing takes an Action on Mammals – say, action1 above, which prints the Latin name of any Animal, and therefore can do so to any Mammal – and then invokes *that* action on a new Giraffe.

Clearly the type parameter A is used in an *input* position in the definition of Meta\<A\>, so we should be able to make it *contravariant*, right? Suppose we did so. That would mean that this assignment would be legal:

 

Meta\<Tiger\> meta2 = meta1; // would be legal if Meta were contravariant in its parameter

But that means that this would then be legal:

 

Action\<Tiger\> action3 = tiger=\>{ tiger.Growl(); };  
meta2(action3);

Follow the logic through and you’ll see that we end up calling (new Giraffe()).Growl(), which is clearly a violation of both the type system and of nature’s laws.

Thus Meta\<A\> *cannot* be *contravariant* in A. It can however be *covariant*:

 

Meta\<Animal\> meta3 = meta1; // legal if Meta were covariant

Now everything works. meta3 takes an Action on Animals and passes a Giraffe to that Action. We’re all good.

Contravariance is tricky; the fact that it *reverses* the bigger/smaller relationship between types means that a type parameter used in a "doubly contravariant" position (being an input of Action, which is itself an input of Meta) becomes *covariant*. The second reversal undoes the first.

**We do not expect that most people will use variance in this manner**; rather, we expect that people will almost always use covariant parameters in output positions and contravariant parameters in input positions. As we’ll see a bit later in this series, whether this expectation is reasonable or not would influence the syntax we might choose were we to add variance to a hypothetical future version of C\#.

Next time: we’ll leave delegates behind and talk about variance in interfaces.

