# Method Hiding Apologia

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/21/2008 10:34:00 AM

-----

Here's some back-and-forth from an email conversation I had with a user a while back.

**Why should one avoid method hiding? **

If there were no advantages and only disadvantages then we would not have added it to the language in the first place. C\# implements hiding because hiding is frequently useful.   
   
I therefore deny the premise of the question. One should not avoid method hiding if it is the right thing to do.

**But I find method hiding confusing. It lets derived types appear to break the contracts of base types. If a derived type D hides a method M on base class B because D.M does something different than B.M, shouldn't it have a different name?**

That sounds remarkably like a good answer to your original question.

However, method hiding is for exactly those times when you need to have two things to have the same name but different behaviour.  
   
Obviously that is not a pleasant situation to be in. I agree with you that it is almost always preferable to have different things have different names. However, I can think of a few situations in which it's desirable. Consider this real-world example:  
   
interface IEnumerable\<T\> : IEnumerable {  
  new IEnumerator\<T\> GetEnumerator();  
}  
   
Is there a better name than GetEnumerator? That's what it does: it gets an enumerator. And in this case, you want to suppress the usage of the base type's non-generic GetEnumerator as much as possible; this version is intended to fully replace the non-generic version.

Of course, if C\# had method return type covariance, this would not have to be a new method. That gives us a larger good reason for method hiding; it allows for something like return type covariance in a language that does not have such a feature.

**I suppose that makes sense. Could you give some other examples of valid usage?**

Sure, but I will have to digress in a prolix manner first.

We want to write computer programs which solve real-world problems, and therefore we want to design languages which enable developers to model their real-world problems in natural, flexible and intuitive ways. We want to take problem solving techniques which work well in the non-computer realm and enable developers to use the real-world intuitions and skills they’ve learned in their program.

One of the techniques that we developed millennia ago to solve problems is organization of objects into hierarchies. Hierarchies are often imperfect – there are the occasional platypuses which crop up and resist easy classification. But hierarchies are such a powerful and useful tool that we use them, despite their flaws, to help organize the world’s data and solve real problems.

Thus, it should be clear that class-based inheritance exists in programming languages because we wish to enable developers to naturally and easily write programs which model problems solved by hierarchies.

So here’s the rub: in a world where objects fit into a hierarchy, who gets to decide how to manipulate that object based on its position in the hierarchy?  Does the object get to decide (at runtime), or does the code doing the manipulation get to decide (at compile time)? 

The former is called “virtual dispatch”, the latter “non-virtual dispatch”.  Which you choose to use depends entirely upon the problem being modeled. It’s not like one of them is absolutely morally better than the other. The better one is the one which models the real-world problem better.

Many problems – probably the majority of problems in this space – are best modeled by letting the object decide how it is to be treated.  When you call Feed on an instance of Animal, let the instance decide what happens based on whether the instance is a Squid or a Zebra. Do a virtual dispatch at runtime. But it would be an error to say that because most problems are modeled this way, that the language ought not to allow modeling the problem any other way. Sometimes it is best for the caller to decide how the object is treated, because the caller has more information.

For example, when I was a teenager my father owned a restaurant. This was at the time that the Conservative government in Canada introduced a highly unpopular value-added sales tax on goods and services called, unimaginatively enough, the Goods and Services Tax.  The GST rules were roundly criticized as being insanely complicated. No government wants to be known as “the government that increased the price of food for poor people”, so grocery items were exempted from the GST. But what qualified as a “grocery item”? That had a whole other complex set of rules.

A cake sold in a grocery store, no GST. The exact same cake, sliced and plated in my father’s restaurant, GST. The exact same cake, NOT sliced, sold whole in the restaurant, no GST. A muffin in a grocery store, no GST. The same muffin in my father’s restaurant, GST. A box of six muffins sold in the restaurant, no GST.

I gather that in the last twenty years the tax has been rationalized somewhat. That’s not my point. My point is that this is a case where what the object is (virtual dispatch) is only one factor in how it behaves with respect to taxes. An equally important factor is how the object is being classified right now (non-virtual dispatch).

How might we model this in a programming language? There are lots of ways, and we could certainly argue about which is best. C\# allows you the flexibility to come up with a variety of different designs and decide for yourself which models your problem best. One way in which we could reasonably model this problem would be to have a hierarchy:

 

abstract class Food {  
    public decimal TaxRate { get { return 7.0m;} }  
}  
abstract class Grocery : Food {  
    new public decimal TaxRate { get { return 0.0m; } }  
}  
class Cake : Grocery {  
    new public decimal TaxRate { get { return 7.0m; } }  
}

And so on. We could continue to complexify this to model the tax rules better. Now when I have a variable of type Cake, it’s tax rate is 7%. When I have the same cake stuck into a variable of type Grocery, its tax rate is 0%. That is, what the object “really” is at runtime is less relevant to the problem at hand then how I am presently classifying it.

One might make the argument that this is a violation of the Single Responsibility Principle, that the tax logic should be in a class of its own, that groceryness should be a flag and not a part of the type system, that really what we need is both a Food and a TransactionContext hierarchy, blah blah blah. I am sure that we could come up with a completely different hierarchy of objects which did not require hiding, and that other hierarchy might even be “better” in some ways. We seem to be badly conflating mechanism with policy here, which is worrisome.

But that's not my point. My point is that in C\# we want to give you the flexibility to model problems this way if you choose to, if you believe that this is the best way to model them. And I think that’s a good thing.

I wrote a couple of articles about how we designed this same feature into JScript .NET. It is a language greatly complicated by its dynamic nature, so the design decisions became correspondingly harder. See

<http://blogs.msdn.com/ericlippert/archive/2004/06/07/150367.aspx>  
<http://blogs.msdn.com/ericlippert/archive/2004/06/08/151209.aspx>

The comments are particularly useful in these articles as well.

