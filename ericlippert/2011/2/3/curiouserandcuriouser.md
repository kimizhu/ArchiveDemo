# Curiouser and curiouser

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/3/2011 6:55:00 AM

-----

Here's a pattern you see all the time in C\#:

 

class Frob : IComparable\<Frob\>

At first glance you might ask yourself why this is not a "circular" definition; after all, you're not allowed to say "class Frob : Frob"(\*). However, upon deeper reflection that makes perfect sense; a Frob is something that can be compared to another Frob. There's not actually a real circularity there.

This pattern can be genericized further:

 

class SortedList\<T\> where T : IComparable\<T\>

Again, it might seem a bit circular to say that T is constrained to something that is in terms of T, but actually this is just the same as before. T is constrained to be something that can be compared to T. Frob is a legal type argument for a SortedList because one Frob can be compared to another Frob.

But this really hurts my brain:

class Blah\<T\> where T : Blah\<T\>

That appears to be circular in (at least) two ways. Is this really legal?

Yes it is legal, and it does have some legitimate uses. I see this pattern rather a lot(\*\*). However, I personally don't like it and I discourage its use.

This is a C\# variation on what's called the [Curiously Recurring Template Pattern](http://en.wikipedia.org/wiki/Curiously_recurring_template_pattern) in C++, and I will leave it to my betters to explain its uses in that language. Essentially the pattern in C\# is an attempt to *enforce* the usage of the CRTP.

So, why would you want to do that, and why do I object to it?

One reason why people want to do this is to enforce a particular constraint in a type hierarchy. Suppose we have

 

abstract class Animal  
{  
    public virtual void MakeFriends(Animal animal);  
}

But that means that a Cat can make friends with a Dog, and [that would be a crisis of Biblical proportions](https://www.youtube.com/watch?v=O3ZOKDmorj0)\! (\*\*\*) What we want to say is

 

abstract class Animal  
{  
    public virtual void MakeFriends(THISTYPE animal);  
}

so that when Cat overrides MakeFriends, it can only override it with Cat.

Now, that immediately presents a problem in that we've just violated the [Liskov Substitution Principle](http://en.wikipedia.org/wiki/Liskov_substitution_principle). We can no longer call a method on a variable of the abstract base type and have any confidence that type safety is maintained. Variance on formal parameter types has to be contravariance, not covariance, for it to be typesafe. And moreover, we simply don't have that feature in the CLR type system.

But you can get close with the curious pattern:

 

abstract class Animal\<T\> where T : Animal\<T\>  
{  
    public virtual void MakeFriends(T animal);  
}

class Cat : Animal\<Cat\>  
{  
  public override void MakeFriends(Cat cat) {}  
}

and hey, we haven't violated the LSP and we have guaranteed that a Cat can only make friends with a Cat. Beautiful.

Wait a minute... did we really guarantee that?

 

class EvilDog : Animal\<Cat\>  
{  
  public override void MakeFriends(Cat cat) { }  
}

We have not guaranteed that a Cat can only make friends with a Cat; an EvilDog can make friends with a Cat too. The constraint only enforces that the type argument to Animal be good; how you use the resulting valid type is entirely up to you. You can use it for a base type of something else if you wish.

So that's one good reason to avoid this pattern: because it doesn't actually enforce the constraint you think it does. Everyone has to play along and agree that they'll use the curiously recurring pattern the way it was *intended* to be used, rather than the evil dog way that it *can* be used.

The second reason to avoid this is simply because it [bakes the noodle](https://www.youtube.com/watch?v=kWVWNri4IFM) of anyone who reads the code. When I see List\<Giraffe\> I have a very clear idea of what the relationship is between the "List" part -- it means that there are going to be operations that add and remove things -- and the "Giraffe" part -- those operations are going to be on Giraffes. When I see "FuturesContract\<T\> where T : LegalPolicy" I understand that this type is intended to model a legal contract about a transaction in the future which has some particular controlling legal policy. But when I read "Blah\<T\> where T : Blah\<T\>" I have no *intuitive* idea of what the intended relationship is between Blah and any particular T. *It seems like an abuse of a mechanism rather than the modeling of a concept from the program's "business domain".*

All that said, in practice there are times when using this pattern really does pragmatically solve problems in ways that are hard to model otherwise in C\#; it allows you to do a bit of an end-run around the fact that we don't have covariant return types on virtual methods, and other shortcomings of the type system. That it does so in a manner that does not, strictly speaking, enforce every constraint you might like is unfortunate, but in realistic code, usually not a problem that prevents shipping the product.

**My advice is to think very hard before you implement this sort of curious pattern in C\#; do the benefits to the customer really outweigh the costs associated with the mental burden you're placing on the code maintainers?**

-----

(\*) Due to an unintentional omission, some past editions of the C\# specification actually did not say that this was illegal\! However, the compiler has always enforced it. In fact, the compiler has over-enforced it, sometimes accidentally catching non-cycles and marking them as cycles.

(\*\*) Most frequently in emails asking "is this really legal?"

(\*\*\*) [Mass hysteria\!](http://blogs.msdn.com/b/ericlippert/archive/2007/05/07/human-sacrifice-dogs-and-cats-living-together-mass-hysteria-and-thread-model-errors.aspx)

