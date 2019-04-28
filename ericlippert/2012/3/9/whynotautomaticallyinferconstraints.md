# Why not automatically infer constraints?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/9/2012 11:09:00 AM

-----

UPDATE: Whoops\! I accidentally set a draft of this article to automatically publish on a day that I was away on vacation. The fact that it was (1) not purple and (2) introduced the topic and then stopped in mid-sentence were both clues that this was an unfinished edit. Sorry about that; I thought I had set it to \*not\* publish on Monday. I am not so good with these computer thingies apparently.

I spent the weekend not thinking about computers by lying beside a pool in Palm Springs, which I can definitively report is an awesome way to spend a rainy weekend in Seattle. I can also definitely report that Peter Frampton has lost almost all his hair and absolutely none of his talent; the man is amazing. If you like 1970's hard rock guitar solos, you've got [just a couple more weeks](http://frampton.com/live/) to hear him perform all of Frampton Comes Alive.

Right, let's actually finish off that article then:

\--------------

Suppose you have a generic base type with a constraint:

class Bravo\<T\> where T : IComparable\<T\> { ... }

If you make a generic derived class in the obvious way:

class Delta\<U\> : Bravo\<U\> { ... }

then the C\# compiler gives you an error:

error CS0314: The type 'U' cannot be used as type parameter 'T' in the generic type or method 'Bravo\<T\>'. There is no boxing conversion or type parameter conversion from 'U' to 'System.IComparable\<U\>'.

Which seems reasonable; every construction of Bravo\<T\> is required to meet the constraints on T and we have no evidence whatsoever that the type supplied for U will meet those constraints.

But that's only one way of looking at the problem; another way of looking at it is that we *do* have evidence that the person who declared U *expected that U meets the constraints of T*, since they used it as T. Given that evidence one might reasonably then expect the compiler to simply silently place the same constraint upon U. U would then meet the constraint on T; any error then would not be on the declaration of Delta's base class, but rather, upon any code which constructs Delta\<U\> such that Bravo\<T\>'s constraints are violated.

I'm often asked why the compiler does not implement this feature or that feature, and of course the answer is always the same: **because no one implemented it.** Features start off as unimplemented and only become implemented when people spend effort implementing them: no effort, no feature. This is an unsatisfying answer of course, because usually the person asking the question has made the assumption that the feature is so *obviously good* that we need to have had a reason to **not** implement it. I assure you that no, we actually don't need a reason to not implement any feature no matter how obviously good. But that said, it might be interesting to consider what sort of pros and cons we'd consider if asked to implement the "silently put inferred constraints on class type parameters" feature.

As C\# has evolved, clearly more and more features involve the compiler silently making inferences on your behalf: method group conversions, method type inference, implicitly typed locals, implicitly typed arrays, implicitly typed lambdas, and so on, all involve a great deal of inference work by the compiler. You would think we could do the same thing for generic type constraints. However, the problem is not as easy as it looks. The easy case mentioned above is, well, easy. Things quickly get complicated:

class Echo\<W, X\> where W : IComparable\<W\> where X : IComparable\<X\> { }  
class Foxtrot\<Y\> : Echo\<Y, Y\> { }

The supposition here is now that Y must be implicitly constrained to be both IComparable\<Y\> and... IComparable\<Y\>.  OK, that seems reasonable. But what if we then change that to:

class Echo\<W, X\> where W : Foo where X : Bar { }  
class Foxtrot\<Y\> : Echo\<Y, Y\> { }

Suppose Foo and Bar are class types with no relationship between them. Now there is no possible type argument for Y that meets the inferred constraint. Is the compiler now required to tell you that fact? How smart does it have to be about that?

Let's make this a bit more complicated. I'll just make something up off the top of my head:

class Golf\<T\> where T : Hotel\<T\> {}  
class Hotel\<U\> : Golf\<Indigo\<U\>\> where U : IJuliet\<Hotel\<U\>\> {}  
class Indigo\<V\> : Hotel\<Golf\<V\>\> where V : IKilo\<V\> {}  
interface IJuliet\<W\> {}  
interface IKilo\<X\>

What are the constraints that we have to infer? Well for T to be a Hotel\<T\>, T has got to be an IJuliet\<Hotel\<T\>\>, so we should add that constraint. But in order to be an IJuliet\<Hotel\<T\>\>, T has got to meet the constraints on Hotel\<T\>... oh, wait, we already added that constraint. Looks like our constraint adder is going to have to be resistant to cycles. But wait, do we have \*all\* the constraints on Hotel\<T\>? So far we have only evaluated the \*stated\* constraints. U in Hotel\<U\> not only has to be an IJuliet\<Hotel\<U\>\>, it also has to be a legal Indigo\<U\>, which means it needs to be an IKilo\<U\>, which means we should add a constraint to T that T be IKilo\<T\>.

And so on. I'm not going to go through a full analysis of this thing. The point is, the analysis is complicated, the analysis may go into infinite loops very easily, and it is not at all clear when you know that you've worked out the full set of type constraints when types refer to each other in arbitrary ways. (And we haven't even considered what happens if the interfaces are covariant or contravariant\!) I don't like adding inference algorithms to the compiler that require cycle detection and do potentially unbounded amounts of work. The possibility of getting the algorithm wrong is very real. Even if we get the algorithm right, the odds that the implementation will be buggy is very high. If we add this feature then the compiler needs to be able to get the right answer all the time, and to give a sensible error message that helps the user if there is no right answer. Both problems seem very difficult.

Moreover, why are we stopping with base types? It seems like if we're going to do the feature of inferring constraints, then we ought to consume information about the entire class, not just the base types:

class Mike\<T\> where T : November {}  
class Oscar\<V\> { Mike\<V\> bv; }

Should we deduce that V must be November because Oscar\<V\> has a field of type Mike\<V\>? I don't see why the base types are any more special than the field types. What if Oscar\<V\> had used Mike\<V\> as a return type of a method? Or a parameter type? Or as the type of a local variable? Where does it stop? Basically, the feature is too much pain for too little gain. When you construct a generic, you are the one required to supply a proof to the compiler that you have satisfied the constraints. C\# is not a "figure out what the developer meant to say and say it for them" language; it's a "tell the developer when they have supplied too little information" language.

