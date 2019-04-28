<div id="page">

# So many interfaces\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/4/2011 8:14:00 AM

-----

<div id="content">

<div class="mine">

Today, another question from [StackOverflow](http://stackoverflow.com/questions/4817369/), and again, presented as a dialogue as is my wont.

> **<span style="color: #000000" color="#000000">The </span>**[**<span style="color: #000000" color="#000000">MSDN documentation for List\<T\></span>**](http://msdn.microsoft.com/en-us/library/6sh2ey19.aspx#Y159)**<span style="color: #000000" color="#000000"> says that the class is declared as </span>**

<span class="code">public class List\<T\> : IList\<T\>, ICollection\<T\>, IEnumerable\<T\>,  
                       </span><span class="code">IList, ICollection, IEnumerable</span>

> **<span style="color: #000000" color="#000000">Does List\<T\> really implement all those interfaces?</span>**

Yes.

> **<span style="color: #333333" color="#333333">Why so many interfaces?</span>**

Because when an interface like <span class="code">IList\<T\></span> inherits from an interface like <span class="code">IEnumerable\<T\></span> then implementers of the more derived interface are required to also implement the less derived interface. That's what interface inheritance means; if you fulfill the contract of the more derived type then you are required to also fulfill the contract of the less derived type.

> **<span style="color: #333333" color="#333333">So a class or struct is required to implement all the methods of all the interfaces in the transitive closure of its base interfaces?</span>**

Exactly.

> <span style="color: #333333" color="#333333">**Is a class (or struct) that implements a more-derived interface also *required* to state in its base type list that it is implementing all of those less-derived interfaces?**</span>

No.

> <span style="color: #333333" color="#333333">**Is the class required to *not* state it?**</span>

No.

> <span style="color: #333333" color="#333333">**So it's *optional* whether the less-derived implemented interfaces are stated in the base type list?**</span>

Yes.

> <span style="color: #333333" color="#333333">**Always?**</span>

Almost always:

<span class="code">interface I1 {}  
interface I2 : I1 {}  
interface I3 : I2 {}</span>

It is optional whether <span class="code">I3</span> states that it inherits from <span class="code">I1</span>.

<span class="code">class B : I3 {}</span>

Implementers of <span class="code">I3</span> are required to implement <span class="code">I2</span> and <span class="code">I1</span>, but they are not required to state explicitly that they are doing so. It's optional.

<span class="code">class D : B {}</span>

Derived classes are not required to re-state that they implement an interface from their base class, but are permitted to do so. (This case is special; see below for more details.)

<span class="code">class C\<T\> where T : I3  
{  
  public virtual void M\<U\>() where U : I3 {}  
}</span>

Type arguments corresponding to <span class="code">T</span> and <span class="code">U</span> are required to implement <span class="code">I2</span> and <span class="code">I1</span>, but it is optional for the constraints on <span class="code">T</span> or <span class="code">U</span> to state that.

It is always optional to re-state any base interface in a partial class:

<span class="code">partial class E : I3 {}  
partial class E {}</span>

The second half of <span class="code">E</span> is permitted to state that it implements <span class="code">I3</span> or <span class="code">I2</span> or <span class="code">I1</span>, but not required to do so.

> <span style="color: #333333" color="#333333">**OK, I get it; it's optional. Why would anyone unnecessarily state a base interface?**</span>

Perhaps because they believe that doing so makes the code easier to understand and more self-documenting.

Or, perhaps the developer wrote the code as

<span class="code">interface I1 {}  
interface I2 {}  
interface I3 : I1, I2 {}</span>

and the realized, oh, wait a minute, <span class="code">I2</span> should inherit from <span class="code">I1</span>. **Why should making that edit then require the developer to go back and change the declaration of <span class="code">I3</span> to *not* contain explicit mention of <span class="code">I1</span>?** I see no reason to force developers to *remove* redundant information.

> <span style="color: #333333" color="#333333">**Aside from being easier to read and understand, is there any *technical* difference between stating an interface explicitly in the base type list and leaving it unstated but implied?**</span>

Usually no, but there can be a subtle difference in one case. Suppose you have a derived class <span class="code">D</span> whose base class <span class="code">B</span> has implemented some interfaces. <span class="code">D</span> automatically implements those interfaces via <span class="code">B</span>. If you restate the interfaces in <span class="code">D</span>'s base class list then the C\# compiler will do an **interface reimplementation**. The details are a bit subtle; if you are interested in how this works then I recommend a careful reading of section 13.4.6 of the C\# 4 specification. Basically the idea is that the compiler "starts over" and figures out which methods implement which interfaces.

UPDATE: [See this follow-up article for details of interface re-implementation.](http://blogs.msdn.com/b/ericlippert/archive/2011/12/08/so-many-interfaces-part-two.aspx)

> **<span style="color: #333333" color="#333333">Does the List\<T\> source code actually state all those interfaces?</span>**

No. The actual source code says

<span class="code">public class List\<T\> : IList\<T\>, System.Collections.IList</span>

> <span style="color: #333333" color="#333333">**Why does MSDN have the full interface list but the real source code does not?**</span>

Because MSDN is *documentation*; it's supposed to give you as much information as you might want. It is much more clear for the documentation to be complete all in one place than to make you search through ten different pages to find out what the full interface set is.

> <span style="color: #333333" color="#333333">**Why do tools like Reflector or the object browser show the whole list?**</span>

Those tools do not have the source code. They only have metadata to work from. Since putting in the full list is optional, the tool has no idea whether the original source code contains the full list or not. It is better to err on the side of more information. Again, the tool is attempting to help you by *showing you more information* rather than *hiding information you might need*.

> <span style="color: #333333" color="#333333">**I noticed that <span class="code">IEnumerable\<T\></span> inherits from <span class="code">IEnumerable</span> but <span class="code">IList\<T\></span> does not inherit from <span class="code">IList</span>. What's up with that?** </span>

The same reason why <span class="code">IEnumerable\<T\></span> can be made covariant in <span class="code">T</span> but <span class="code">IList\<T\></span> cannot. A sequence of integers can be treated as a sequence of objects, by boxing every integer as it comes out of the sequence. But a read-write list of integers cannot be treated as a read-write list of objects, because you can put a string into a read-write list of objects. An <span class="code">IEnumerable\<T\></span> can easily fulfill the contract of <span class="code">IEnumerable</span> just by adding a boxing helper method. But <span class="code">IList\<T\></span> is not required to fulfill the whole contract of <span class="code">IList</span>, so it does not inherit from it.

> <span style="color: #333333" color="#333333">**Why then does <span class="code">List\<T\></span> implement <span class="code">IList</span>?**</span>

It is a bit odd, since <span class="code">List\<T\></span> for any type other than object does not fulfill the full contract of <span class="code">IList</span>. It's probably to make it easier on people who are updating old C\# 1.0 code to use generics; those people were probably already ensuring that only the right types got into their lists. And most of the time when you're passing an <span class="code">IList</span> around, it is so the callee can get by-index access to the list, not so that it can add new items of arbitrary type.

</div>

</div>

</div>

