# So many interfaces\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/4/2011 8:14:00 AM

-----

Today, another question from [StackOverflow](http://stackoverflow.com/questions/4817369/), and again, presented as a dialogue as is my wont.

> **The **[**MSDN documentation for List\<T\>**](http://msdn.microsoft.com/en-us/library/6sh2ey19.aspx#Y159)** says that the class is declared as **

public class List\<T\> : IList\<T\>, ICollection\<T\>, IEnumerable\<T\>,  
                       IList, ICollection, IEnumerable

> **Does List\<T\> really implement all those interfaces?**

Yes.

> **Why so many interfaces?**

Because when an interface like IList\<T\> inherits from an interface like IEnumerable\<T\> then implementers of the more derived interface are required to also implement the less derived interface. That's what interface inheritance means; if you fulfill the contract of the more derived type then you are required to also fulfill the contract of the less derived type.

> **So a class or struct is required to implement all the methods of all the interfaces in the transitive closure of its base interfaces?**

Exactly.

> **Is a class (or struct) that implements a more-derived interface also *required* to state in its base type list that it is implementing all of those less-derived interfaces?**

No.

> **Is the class required to *not* state it?**

No.

> **So it's *optional* whether the less-derived implemented interfaces are stated in the base type list?**

Yes.

> **Always?**

Almost always:

interface I1 {}  
interface I2 : I1 {}  
interface I3 : I2 {}

It is optional whether I3 states that it inherits from I1.

class B : I3 {}

Implementers of I3 are required to implement I2 and I1, but they are not required to state explicitly that they are doing so. It's optional.

class D : B {}

Derived classes are not required to re-state that they implement an interface from their base class, but are permitted to do so. (This case is special; see below for more details.)

class C\<T\> where T : I3  
{  
  public virtual void M\<U\>() where U : I3 {}  
}

Type arguments corresponding to T and U are required to implement I2 and I1, but it is optional for the constraints on T or U to state that.

It is always optional to re-state any base interface in a partial class:

partial class E : I3 {}  
partial class E {}

The second half of E is permitted to state that it implements I3 or I2 or I1, but not required to do so.

> **OK, I get it; it's optional. Why would anyone unnecessarily state a base interface?**

Perhaps because they believe that doing so makes the code easier to understand and more self-documenting.

Or, perhaps the developer wrote the code as

interface I1 {}  
interface I2 {}  
interface I3 : I1, I2 {}

and the realized, oh, wait a minute, I2 should inherit from I1. **Why should making that edit then require the developer to go back and change the declaration of I3 to *not* contain explicit mention of I1?** I see no reason to force developers to *remove* redundant information.

> **Aside from being easier to read and understand, is there any *technical* difference between stating an interface explicitly in the base type list and leaving it unstated but implied?**

Usually no, but there can be a subtle difference in one case. Suppose you have a derived class D whose base class B has implemented some interfaces. D automatically implements those interfaces via B. If you restate the interfaces in D's base class list then the C\# compiler will do an **interface reimplementation**. The details are a bit subtle; if you are interested in how this works then I recommend a careful reading of section 13.4.6 of the C\# 4 specification. Basically the idea is that the compiler "starts over" and figures out which methods implement which interfaces.

UPDATE: [See this follow-up article for details of interface re-implementation.](http://blogs.msdn.com/b/ericlippert/archive/2011/12/08/so-many-interfaces-part-two.aspx)

> **Does the List\<T\> source code actually state all those interfaces?**

No. The actual source code says

public class List\<T\> : IList\<T\>, System.Collections.IList

> **Why does MSDN have the full interface list but the real source code does not?**

Because MSDN is *documentation*; it's supposed to give you as much information as you might want. It is much more clear for the documentation to be complete all in one place than to make you search through ten different pages to find out what the full interface set is.

> **Why do tools like Reflector or the object browser show the whole list?**

Those tools do not have the source code. They only have metadata to work from. Since putting in the full list is optional, the tool has no idea whether the original source code contains the full list or not. It is better to err on the side of more information. Again, the tool is attempting to help you by *showing you more information* rather than *hiding information you might need*.

> **I noticed that IEnumerable\<T\> inherits from IEnumerable but IList\<T\> does not inherit from IList. What's up with that?** 

The same reason why IEnumerable\<T\> can be made covariant in T but IList\<T\> cannot. A sequence of integers can be treated as a sequence of objects, by boxing every integer as it comes out of the sequence. But a read-write list of integers cannot be treated as a read-write list of objects, because you can put a string into a read-write list of objects. An IEnumerable\<T\> can easily fulfill the contract of IEnumerable just by adding a boxing helper method. But IList\<T\> is not required to fulfill the whole contract of IList, so it does not inherit from it.

> **Why then does List\<T\> implement IList?**

It is a bit odd, since List\<T\> for any type other than object does not fulfill the full contract of IList. It's probably to make it easier on people who are updating old C\# 1.0 code to use generics; those people were probably already ensuring that only the right types got into their lists. And most of the time when you're passing an IList around, it is so the callee can get by-index access to the list, not so that it can add new items of arbitrary type.

