# Putting a base in the middle

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/29/2010 6:16:00 AM

-----

Here’s a crazy-seeming but honest-to-goodness real customer scenario that got reported to me recently. There are three DLLs involved, Alpha.DLL, Bravo.DLL and Charlie.DLL. The classes in each are:

 

public class Alpha // In Alpha.DLL  
{  
  public virtual void M()  
  {  
    Console.WriteLine("Alpha");  
  }  
}

public class Bravo: Alpha // In Bravo.DLL  
{  
}

public class Charlie : Bravo // In Charlie.DLL  
{  
  public override void M()  
  {  
    Console.WriteLine("Charlie");  
    base.M();  
  }  
}

Perfectly sensible. You call M on an instance of Charlie and it says “Charlie / Alpha”.

Now the vendor who supplies Bravo.DLL ships a new version which has this code:

 

public class Bravo: Alpha  
{  
  public override void M()  
  {  
    Console.WriteLine("Bravo");  
    base.M();  
  }  
}

The question is: *what happens if you call Charlie.M **without recompiling Charlie.DLL**, but you are loading the new version of Bravo.DLL?*

The customer was quite surprised that the output is still “Charlie / Alpha”, not “Charlie / Bravo / Alpha”.

This is a new twist on the [brittle base class failure](http://blogs.msdn.com/ericlippert/archive/tags/Brittle+Base+Classes/default.aspx); at least, it’s new to me.

**Customer: What’s going on here?**

When the compiler generates code for the base call, it looks at all the metadata and sees that the nearest valid method that the base call can be referring to is Alpha.Foo. So we generate code that says “make a non-virtual call to Alpha.Foo”. That code is baked into Charlie.DLL and it has the same semantics no matter what Bravo.DLL says. It calls Alpha.Foo.

**Customer: You know, if you generated code that said “make a non-virtual call to Bravo.Foo”, the CLR will fall back to calling Alpha.Foo if there is no implementation of Bravo.Foo.**

No, I didn’t know that actually. I’m slightly surprised that this doesn’t produce a verification error, but, whatever. Seems like a plausible behaviour, albeit perhaps somewhat risky. A quick look at the documented semantics of the call instruction indicates that this is the by-design behaviour, so it would be legal to do so.

**Customer: Why doesn’t the compiler generate the call as a call to Bravo.Foo? Then you get the right semantics in my scenario\!**

Essentially what is happening here is the compiler is generating code on the basis of *today's* static analysis, not on the basis of what the world *might* look like at runtime in an unknown future. When we generate the code for the base call we assume that there are not going to be changes in the base class hierarchy after compilation. That seemed at the time to be a reasonable assumption, though I can see that in your scenario, arguably it is not. As it turns out, there are two reasons to do it the current way. The first is philosophical and apparently unconvincing. The second is practical. **Customer: What’s the philosophical justification?** There are two competing "mental models" of what "base.Foo" means. The mental model that matches what the compiler currently implements is “a base call is a *non-virtual* call to the *nearest method* on any base class, *based entirely on information known at compile time.”* Note that this matches exactly what we mean by "non-virtual call". An early-bound call to a non-virtual method is always a call to a *particular method identified at compile time.* By contrast, a virtual method call is based at least in part on runtime analysis of the type hierarchy. More specifically, a virtual method identifies a "slot" at compile time but not the "contents" of that slot. The "contents" – the actually method to call – is identified at runtime based on what the runtime type of the receiver stuffed into the virtual method slot. Your mental model is “a base call is a *virtual* call to the *nearest method* on any base class, based on both information *known at runtime* about the actual class hierarchy of the receiver, and information known *at compile time* about the compile-time type of the receiver.” In your model the call is not actually virtual, because it is not based upon the contents of a virtual slot of the receiver. But neither is it entirely based on the compile-time knowledge of the type of the receiver\! It's based on a combination of the two. Basically, it’s *what would have been the non-virtual call in the counterfactual world where the compiler had been given correct information about what the types actually would look like at runtime.* A developer who has the former mental model (like, say, me) would be deeply surprised by your proposed behavior. If the developer has classes Giraffe, Mammal and Animal, Giraffe overrides virtual method Animal.Feed, and the developer says base.Feed in Giraffe, then the developer is thinking either like me:

> I specifically wish Animal.Feed to be called here; if at runtime it turns out that evil hackers have inserted a method Mammal.Feed that I did not know about at compile time, I still want Animal.Feed to be called. I have compiled against Animal.Feed, I have tested against that scenario, and that call is precisely what I expect to happen. **A base call gives me 100% of the safe, predictable, understandable, non-dynamic, testable behavior of any other non-virtual call. I rely upon those invariants to keep my customer's data secure.**

Basically, this position is "I trust only what I can see when I wrote the code; any other code might not do what I want safely or correctly". Or like you:

> I need the base class to do some work for me. I want something on some base class to be called. Animal.Feed or Mammal.Feed, I don't care, just pick the best one - whichever one happens to be "most derived" in some future version of the world - by doing that analysis at runtime. **In exchange for the flexibility of being able to hot-swap in new behavior by changing the implementation of my base classes without recompiling my derived classes, I am willing to give up safety, predictability, and the knowledge that what runs on my customer's machines is what I tested.**

Basically, this position is "I trust that the current version of my class knows how to interpret my request and will do so safely and correctly, even if I've never once tested that." Though I understand your point of view, I’m personally inclined to do things the safe, boring and sane way rather than the flexible, dangerous and interesting way. However, based on the several dozen comments on the first version of this article, and my brief poll of other members of the C\# compiler team, I am in a small minority that believes that the first mental model is the more sensible one.

**Customer: The philosophical reason is unconvincing; I see a base call as meaning “call the nearest thing in the virtual hierarchy”. What’s the practical concern?**

In the autumn of 2000, during the development of C\# 1.0, the behaviour of the compiler was as you expect: we would generate a call to Bravo.M and allow the runtime to resolve that as either a call to Bravo.M if there is one or to Alpha.M if there is not. My predecessor Peter Hallam then discovered the following case. Suppose the new hot-swapped Bravo.DLL is now:

 

public class Bravo: Alpha  
{  
  new private void M()  
  {  
    Console.WriteLine("Bravo");  
  }  
}

Now what happens? Bravo has added a private method, and one of our design principles is that **private methods are invisible implementation details**; they do not have *any* effect on the surrounding code that cannot see them. If you hot-swap in this code and the call in Charlie is realized as a call to Bravo.M then **this crashes the runtime**. The base call resolves as a call to a private method from outside the method, which is not legal. **Non-virtual calls do matching by signature, not by virtual slot.**

The CLR architects and the C\# architects considered many possible solutions to this problem, including adding a new instruction that would match by slot, changing the semantics of the call instruction, changing the meaning of "private", implementing name mangling in the compiler, and so on. The decision they arrived at was that all of the above were insanely dangerous considering how late in the ship cycle it was, how unlikely the scenario is, and the fact that this would be enabling a scenario which is directly contrary to good sense; **if you change a base class then you should recompile your derived classes.** We don't want to be in the business of making it easier to do something **dangerous and wrong**.

So they punted on the issue. The C\# 1.0 compiler apparently did it the way you like, and generated code that sometimes crashed the runtime if you introduced a new private method: the original compilation of Charlie calls Bravo.M, even if there is no such method. If later there turns out to be an inaccessible one, it crashes. **If you recompile Charlie.DLL, then the compiler notices that there is an intervening private method which will crash the runtime, and generates a call to Alpha.M.**

This is far from ideal. The compiler is designed so that for performance reasons it does not load the potentially hundreds of millions of bytes of metadata about private members from referenced assemblies; now we have to load at least some of that. Also, this makes it difficult to use tools such as ASMMETA which produce "fake" versions of assemblies which are then later replaced with real assemblies. And of course there is always still the crashing scenario to worry about.

The situation continued thusly until 2003, at which point again the C\# team brought this up with the CLR team to see if we could get a new instruction defined, a "basecall" instruction which would provide an exact virtual slot reference, rather than doing a by-signature match as the non-virtual call instruction does now. After much debate it was again determined that **this obscure and dangerous scenario did not meet the bar for making an extremely expensive and potentially breaking change to the CLR.**

Concerned over all the ways that this behaviour was currently causing breaks and poor performance, in 2003 the C\# design team decided to go with the present approach of binding directly to the slot as known at compile time. The team all agreed that the desirable behaviour was to always dynamically bind to the closest base class -- a point which I personally disagree with, but I see their point. But given the costs of doing so safely, and the fact that hot-swapping in new code in the middle of a class hierarchy is not exactly a desirable scenario to support, it's better to sometimes force a recompilation (that you should have done anyways) than to sometimes crash and die horribly.

**Customer: Wow. So, this will never change, right?**

Wow indeed. I learned an awful lot today. One of these days I need to sit down and just read all five hundred pages of the C\# 1.0 and 2.0 design notes.

I wouldn’t expect this to ever change. If you change a base class, recompile your derived classes. That’s the safe thing to do. Do not rely on the runtime fixing stuff up for you when you hot-swap in a new class in the middle of a class hierarchy.

UPDATE: Based on the number of rather histrionic comments I've gotten over the last 24 hours, I think my advice above has been taken rather out of the surrounding context. I'm not saying that every time someone ships a service pack that has a few bug fixes that you are required to recompile all your applications and ship them again. I thought it was clear from the context that what I was saying was that if you depend upon base type which has been updated then:

(1) at the very least **test your derived types with the new base type** -- your derived types are relying on the mechanisms of the base types; when a mechanism changes, you have to re-test the code which relies upon that mechanism.

(2) **if there was a breaking change, recompile, re-test and re-ship the derived type**. And

(3) you might be surprised by what is a breaking change; **adding a new override can potentially be a breaking change in some rare cases**.

I agree that it is unfortunate that adding a new override is in some rare cases a semantically breaking change. I hope you agree that it is also unfortunate that adding a new *private* method was in some rare cases a crash-the-runtime change in C\# 1.0. Which of those evils is the lesser is of course a matter of debate; we had that debate between 2000 and 2003 and I don't think its wise or productive to second-guess the outcome of that debate now.

The simple fact of the matter is that the brittle base class problem is an inherant problem with the object-oriented-programming pattern. **We have worked very hard to design a language which minimizes the likelihood of the brittle base class problem biting people**. And the base class library team works very hard to ensure that **service pack upgrades introduce as few breaking changes as possible** while meeting our other servicing goals, like fixing existing problems. But our hard work only goes so far, and there are more base classes in the world that those in the BCL.

If you find that you are getting bitten by the brittle base class problem a lot, then maybe object oriented programming is not actually the right solution for your problem space; there are other approaches to code reuse and organization which are perfectly valid that do not suffer from the brittle base class problem.

