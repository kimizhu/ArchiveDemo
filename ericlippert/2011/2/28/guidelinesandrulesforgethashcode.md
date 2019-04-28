# Guidelines and rules for GetHashCode

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/28/2011 6:39:00 AM

-----

"[The code is more what you'd call guidelines than actual rules](https://www.youtube.com/watch?v=bplEuBjppTw)" - truer words were never spoken. It's important when writing code to understand what are vague "guidelines" that *should* be followed but can be broken or fudged, and what are crisp "rules" that have serious negative consequences for correctness and robustness. I often get questions about the rules and guidelines for [GetHashCode](http://msdn.microsoft.com/en-us/library/system.object.gethashcode.aspx), so I thought I might summarize them here.

**What is GetHashCode used for?**

It is *by design* useful for only one thing: putting an object in a hash table. Hence the name.

**Why do we have this method on Object in the first place?**

It makes perfect sense that every object in the type system should provide a GetType method; data's ability to describe itself is a key feature of the CLR type system. And it makes sense that every object should have a ToString, so that it is able to print out a representation of itself as a string, for debugging purposes. It seems plausible that objects should be able to compare themselves to other objects for equality. But why should it be the case that every object should be able to hash itself for insertion into a hash table? Seems like an odd thing to require every object to be able to do.

I think if we were redesigning the type system from scratch today, hashing might be done differently, perhaps with an IHashable interface. But when the CLR type system was designed there were no generic types and therefore a general-purpose hash table needed to be able to store any object.

**How do hash tables and similar data structures use GetHashCode?**

Consider a "set" abstract data type. Though there are many operations you might want to perform on a set, the two basic ones are insert a new item into the set, and check to see whether a given item is in the set. We would like these operations to be fast even if the set is large. Consider, for example, using a list as an implementation detail of a set:

 

class Set\<T\>  
{  
  private List\<T\> list = new List\<T\>();  
  
  public void Insert(T item)  
  {  
    if (\!Contains(t))  
      list.Add(item);  
  }  
  
  public bool Contains(T item)  
  {  
    foreach(T member in list)  
      if (member.Equals(item))  
        return true;  
    return false;  
  }  
}

(I've omitted any error checking throughout this article; we probably want to make sure that the item is not null. We probably want to implement some interfaces, and so on. I'm keeping things simple so that we concentrate on the hashing part.)

The test for containment here is linear; if there are ten thousand items in the list then we have to look at all ten thousand of them to determine that the object is not in the list. This does not scale well.

The trick is to trade a small amount of increased memory burden for a huge amount of increased speed. The idea is to make many shorter lists, called "buckets", and then be clever about quickly working out which bucket we're looking at:

 

class Set\<T\>  
{  
  private List\<T\>\[\] buckets = new List\<T\>\[100\];  
  
  public void Insert(T item)  
  {  
    int bucket = GetBucket(item.GetHashCode());  
    if (Contains(item, bucket))  
      return;  
    if (buckets\[bucket\] == null)  
      buckets\[bucket\] = new List\<T\>();  
    buckets\[bucket\].Add(item);  
  }  
  
  public bool Contains(T item)  
  {  
    return Contains(item, GetBucket(item.GetHashCode());  
  }  
  
  private int GetBucket(int hashcode)  
  {  
    unchecked  
    {  
      // A hash code can be negative, and thus its remainder can be negative also.  
      // Do the math in unsigned ints to be sure we stay positive.  
      return (int)((uint)hashcode % (uint)buckets.Length);  
    }  
  }  
  
  private bool Contains(T item, int bucket)  
  {  
    if (buckets\[bucket\] \!= null)  
      foreach(T member in buckets\[bucket\])  
        if (member.Equals(item))  
          return true;  
    return false;  
  }  
}

Now if we have ten thousand items in the set then we are looking through one of a hundred buckets, each with on average a hundred items; the Contains operation just got a hundred times cheaper.

On average.

We hope.

We could be even more clever here; just as a List\<T\> resizes itself when it gets full, the bucket set could resize itself as well, to ensure that the average bucket length stays low. Also, for technical reasons it is often a good idea to make the bucket set length a prime number, rather than 100. There are plenty of improvements we could make to this hash table. But this quick sketch of a naive implementation of a hash table will do for now. I want to keep it simple.

By starting with the position that this code should work, we can deduce what the rules and guidelines must be for GetHashCode:

**Rule: equal items have equal hashes**

If two objects are equal then they must have the same hash code; or, equivalently, if two objects have different hash codes then they must be unequal.

The reasoning here is straightforward. Suppose two objects were equal but had different hash codes. If you put the first object in the set then it might be put into bucket \#12. If you then ask the set whether the second object is a member, it might search bucket \#67, and not find it.

Note that it is NOT a rule that if two objects have the same hash code, then they must be equal. There are only four billion or so possible hash codes, but obviously there are more than four billion possible objects. There are far more than four billion ten-character strings alone. Therefore there must be at least two unequal objects that share the same hash code, by the [Pigeonhole Principle](http://en.wikipedia.org/wiki/Pigeonhole_principle). (\*)

**Guideline: the integer returned by GetHashCode should never change**

Ideally, the hash code of a mutable object should be computed from only fields which cannot mutate, and therefore the hash value of an object is the same for its entire lifetime.

However, this is only an ideal-situation guideline; the actual rule is:

**Rule: the integer returned by GetHashCode must never change while the object is contained in a data structure that depends on the hash code remaining stable**

It is permissible, though dangerous, to make an object whose hash code value can mutate as the fields of the object mutate. If you have such an object and you put it in a hash table then the code which mutates the object and the code which maintains the hash table are required to have some agreed-upon protocol that ensures that the object is not mutated while it is in the hash table. What that protocol looks like is up to you.

If an object's hash code can mutate while it is in the hash table then clearly the Contains method stops working. You put the object in bucket \#5, you mutate it, and when you ask the set whether it contains the mutated object, it looks in bucket \#74 and doesn't find it.

Remember, objects can be put into hash tables in ways that you didn't expect. A lot of the LINQ sequence operators use hash tables internally. Don't go dangerously mutating objects while enumerating a LINQ query that returns them\!

**Rule: Consumers of GetHashCode cannot rely upon it being stable over time or across appdomains**

Suppose you have a Customer object that has a bunch of fields like Name, Address, and so on. If you make two such objects with exactly the same data in two different processes, they do not have to return the same hash code. If you make such an object on Tuesday in one process, shut it down, and run the program again on Wednesday, the hash codes can be different.

This has bitten people in the past. The [documentation](http://msdn.microsoft.com/en-us/library/system.string.gethashcode.aspx) for System.String.GetHashCode notes specifically that two identical strings can have different hash codes in different versions of the CLR, and in fact they do. Don't store string hashes in databases and expect them to be the same forever, because they won't be.

**Rule: GetHashCode must never throw an exception, and must return**

Getting a hash code simply calculates an integer; there's no reason why it should ever fail. An implementation of GetHashCode should be able to handle any legal configuration of the object.

I occasionally get the response "but I want to throw NotImplementedException in my GetHashCode to ensure that the object is never put into a hash table; I don't intend for this object to ever be put into a hash table."  Well, OK, but the last sentence of the previous guideline applies; this means that your object cannot be a result in many LINQ-to-objects queries that use hash tables internally for performance reasons.

Since it doesn't throw an exception, it has to return a value eventually. It's not legal or smart to make an implementation of GetHashCode that goes into an infinite loop.

This is particularly important when hashing objects that might be recursively defined and contain circular references. If hashing object Alpha hashes the value of property Beta, and hashing Beta turns right around and hashes Alpha, then you're going to either loop forever (if you're on an architecture that can optimize tail calls) or run out of stack and crash the process.

**Guideline: the implementation of GetHashCode must be extremely fast**

The whole point of GetHashCode is to optimize a lookup operation; if the call to GetHashCode is slower than looking through those ten thousand items one at a time, then you haven't made a performance gain.

I classify this as a "guideline" and not a "rule" because it is so vague. How slow is too slow? That's up to you to decide.

**Guideline: the distribution of hash codes must be "random"**

By a "random distribution" I mean that if there are commonalities in the objects being hashed, there should not be similar commonalities in the hash codes produced. Suppose for example you are hashing an object that represents the latitude and longitude of a point. A set of such locations is highly likely to be "clustered"; odds are good that your set of locations is, say, mostly houses in the same city, or mostly valves in the same oil field, or whatever. If clustered data produces clustered hash values then that might decrease the number of buckets used and cause a performance problem when the bucket gets really big.

Again, I list this as a guideline rather than a rule because it is somewhat vague, not because it is unimportant. It's very important. But since good distribution and good speed can be opposites, it's important to find a good balance between the two.

I know this from [deep, personal, painful experience](http://blogs.msdn.com/b/ericlippert/archive/2003/09/19/arrrrr-cap-n-eric-be-learnin-about-threadin-the-harrrrd-way.aspx). Over a decade ago I wrote a string hash algorithm for a table used by the msn.com backend servers. I thought it was a reasonably randomly distributed algorithm, but I made a mistake and it was not. It turned out that all of the one hundred thousand strings that are five characters long and contain only numbers were always hashed to one of five buckets, instead of any of the six hundred or so buckets that were available. The msn.com guys were using my table to attempt to do fast lookups of tens of thousands of US postal codes, all of which are strings of five digits. Between that and a threading bug in the same code, I wrecked the performance of an important page on msn.com; this was both costly and embarrassing. Data is sometimes heavily clustered, and a good hash algorithm will take that into account.

In particular, be careful of "xor". It is very common to combine hash codes together by xoring them, but that is not necessarily a good thing. Suppose you have a data structure that contains strings for shipping address and home address. Even if the hash algorithm on the individual strings is really good, if the two strings are frequently the same then xoring their hashes together is frequently going to produce zero. **"xor" can create or exacerbate distribution problems when there is redundancy in data structures.**

**Security issue: if the hashed data can be chosen by an attacker then you might have a problem on your hands**

When I wrecked that page on msn.com it was an accident that the chosen data interacted poorly with my algorithm. But suppose the page was in fact collecting data from users and storing it in a hash table for server-side analysis. If the user is hostile and can deliberately craft huge amounts of data that always hashes to the same bucket then they can mount a denial-of-service attack against the server by making the server waste a lot of time looking through an unbalanced hash table. If that's the situation you are in, consult an expert. It is possible to build hostile-data-resistant implementations of GetHashCode but doing so correctly and safely is a job for an expert in that field.

**Security issue: do not use GetHashCode "off label"**

GetHashCode is designed to do only one thing: balance a hash table. **Do not use it for anything else**. In particular:

\* It does not provide a unique key for an object; [probability of collision is extremely high](http://blogs.msdn.com/b/ericlippert/archive/2010/03/22/socks-birthdays-and-hash-collisions.aspx).  
\* It is not of cryptographic strength, so do not use it as part of a [digital signature](http://blogs.msdn.com/b/ericlippert/archive/2005/10/24/482447.aspx) or as a [password equivalent](http://blogs.msdn.com/b/ericlippert/archive/2005/01/31/363844.aspx)  
\* It does not necessarily have the error-detection properties needed for checksums.

and so on.

Getting all this stuff right is surprisingly tricky.

-----

(\*) If you have ten pigeons kept in nine pigeonholes, then at least one pigeonhole has more than one pigeon in it.

