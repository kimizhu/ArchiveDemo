# Why Can't I Access A Protected Member From A Derived Class, Part Two: Why Can I?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/28/2008 7:30:43 PM

-----

This is a follow-up to [my 2005 post on the same subject](http://blogs.msdn.com/ericlippert/archive/2005/11/09/491031.aspx)  which I believe sets a personal record for the longest time between parts of a series. (Of course, I didn't know it was a series when I started it.) Please read the previous article in this series, as this post assumes knowledge of part one.

.......

OK, now that you've read that, it's clear why you can only access a protected member from an instance of an object known to be of a type at least as derived as the current context. You can therefore deduce the answer to the question asked to me by a (very polite) reader this morning: **Why did this code compile in C\# 2.0 but give an error in C\# 3.0?**

 

public abstract class Item{  
  private Item \_parent;  
  public Item Parent {  
    get { return \_parent; }  
    protected set { \_parent = value; }  
  }  
}  
public class Bag:Item{  
  private List\<Item\> list = new List\<Item\>();  
  public void Add(Item item)  
  {  
    item.Parent = this; // Error in C\# 3.0  
    list.Add(item);   
  }  
}  
public class Torch : Item { }

It compiled in C\# 2.0 because the compiler had a bug. We forgot to enforce the semantics of "protected" access for the property setter. Though the compiler did not generate an error, it did generate code which would not pass the CLR verification check\! The code would run if it happened to be fully trusted, but it was not safe to do so. We fixed the bug in C\# 3.0 and took the breaking change.

This raises a more interesting point though. Now that we correctly prevent you from setting the "parent" reference in this manner, how would you implement the desired pattern? The reader wanted the following conditions to be met:

  - There are two kinds of items -- "leaf" items, and "container" items. Container items may contain either kind of item, so you could have a box containing a bag, which in turn contains a torch.
  - An item may be in a container, and if it is, its parent reference refers to that container.
  - These classes must be extensible by arbitrary third parties.
  - "Unauthorized" code must not be able to muck around with the parenting invariants. 

The reader was attempting to enforce these invariants by making the parent setter protected. But even in a world where it is legal to access a protected member from an arbitrary derived class, that does not ensure that the invariants are maintained\! If you allow any derived class to muck with the parenting, then you are relying upon every third-party derived class to "play nicely" and maintain your invariant. If any of them are buggy or hostile, then who knows what can happen?

When I'm faced with this kind of problem, I try to go back to first principles. Considering each design constraint leads us to an implementation decision which implements that constraint:

  - There are two kinds of abstract items -- items and containers. Therefore, there should be two abstract base classes, not just one.
  - Containers are items. Therefore, the container class should derive from the item class.
  - The parenting invariant must be maintained across all containers. Therefore the invariant implementation must be inside the abstract container base class.
  - Every possible container requires "write" access to the parent state of every possible Item.  We want that access to be as restricted as possible. Ideally, we want it to be private. *The only by-design way to get access to a private is to be inside the class*.

And now a full solution becomes very straightforward:

 

using System;  
using System.Collections.Generic;  
  
public abstract class Item {  
  public Item Parent { get; private set; }  
  public abstract class Container : Item {  
    private HashSet\<Item\> items = new HashSet\<Item\>();  
    public void Add(Item item) {  
      if (item.Parent \!= null)  
        throw new Exception("Item has inconsistent containment.");  
      item.Parent = this;  
      items.Add(item);  
    }  
    public void Remove(Item item) {  
      if (\!Contains(item))  
        throw new Exception("Container does not contain that item.");  
      items.Remove(item);  
      item.Parent = null;  
    }  
    public bool Contains(Item item) {  
      return items.Contains(item);  
    }  
    public IEnumerable\<Item\> Items {  
      // Do not just return items. Then the caller could cast it  
      // to HashSet\<Item\> and then make modifications to your  
      // internal state\! Return a read-only sequence:  
      get {  
        foreach(Item item in items) yield return item;  
      }  
    }  
  }  
}

// These can be in third-party assemblies:  
  
public class Bag : Item.Container { }  
public class Box : Item.Container { }  
public class Torch : Item { }  
public class TreasureMap : Item { }  
  
public class Program {  
  public static void Main() {  
    var map = new TreasureMap();  
    var box = new Box();  
    box.Add(map);  
    var bag = new Bag();  
    bag.Add(box);  
    foreach(Item item in bag.Items)  
      Console.WriteLine(item);  
  }  
}

Pretty slick eh?

A couple questions for you to ponder:

1\) Suppose you were a hostile third party and you wanted to mess up the parenting invariant. Clearly, if you are sufficiently trusted, you can always use private reflection or unsafe code to muck around with the state directly, so that's not a very interesting attack. Any other bright ideas come to mind for ways that this code is vulnerable to tampering?

2\) Suppose you wanted to make this hierarchy an immutable collection, where "Add" and "Remove" returned new collections rather than mutating the existing collection. How would you represent the parenting relationship?

Next time, another oddity involving "protected" semantics. Have a good weekend\!

