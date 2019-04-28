# Read-only and threadsafe are different

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/23/2011 7:47:00 AM

-----

Here's a common problem that we face in the compiler realm all the time: you want to make an efficient immutable lookup table for mapping names to "symbols". This is in a sense the primary problem that the compiler has to solve; someone says "x = y + z;" and we have to figure out what "x", "y" and "z" mean before we can do any more analysis. An obvious way to do that is to figure out all the name-to-symbol mappings for a particular declaration space once, ahead of time, stuff the results into a lookup table, and then use that lookup table during the analysis.

The lookup table can be immutable because once it is built, it's built; no more symbols are going to be added to it. It's going to be used solely as a lookup mechanism. A common and cheap way of doing that is to use what I whimsically call "[popsicle](http://blogs.msdn.com/b/ericlippert/archive/2007/11/13/immutability-in-c-part-one-kinds-of-immutability.aspx)" immutability: the data structure is a fully mutable structure until you freeze it, at which point it becomes an error to attempt to change it. This technique differs markedly from the sort of "persistently re-usable data structure" immutability [we've talked about in the past](http://blogs.msdn.com/b/ericlippert/archive/2007/12/10/immutability-in-c-part-four-an-immutable-queue.aspx), where you want to reuse existing bits of the data structure as you build new, different data structures out of it.

For example, we might write up a really simple little hash table that supports "freezing". Something like this:

 

abstract class Symbol  
{  
    public string Name { get; protected set; }  
}

sealed class SymbolTable  
{  
    private bool frozen = false;

    private class BucketListNode  
    {  
        public Symbol Symbol { get; set; }  
        public BucketListNode Next { get; set; }  
        public BucketListNode Prev { get; set; }  
    }

    private BucketListNode\[\] buckets = new BucketListNode\[17\];  
  
    private int GetBucket(string s)  
    {  
        return Math.Abs(s.GetHashCode()) % buckets.Length;  
    }

    public void Add(Symbol symbol)  
    {  
        // Omitted: error checking for null symbol, null name, duplicated entry, and so on.  
        if (this.frozen)  
            throw new InvalidOperationException();  
        int bucket = GetBucket(symbol.Name);  
        var node = new BucketListNode();  
        node.Symbol = symbol;  
        node.Prev = null;  
        node.Next = buckets\[bucket\];  
        if (node.Next \!= null)  
            node.Next.Prev = node;  
        buckets\[bucket\] = node;  
    }

    public void Freeze()  
    {  
        this.frozen = true;  
    }

    public Symbol Lookup(string name)  
    {  
        // Omitted: error checking  
        int bucket = GetBucket(name);  
        for (var node = buckets\[bucket\]; node \!= null; node = node.Next)  
        {  
            if (node.Symbol.Name == name)  
                return node.Symbol;  
        }  
        return null;  
    }  
}

Very simple. We have an array of seventeen doubly-linked lists. We balance the table by choosing which of the seventeen linked lists to go with by [hashing the name of the symbol](http://blogs.msdn.com/b/ericlippert/archive/2011/02/28/guidelines-and-rules-for-gethashcode.aspx). That way our lookup cost is one-seventeeth the cost of doing a straight lookup in a complete list.

Now of course there are other ways to do this efficiently. We could be sorting the list of symbols and doing a binary search. We could be re-hashing into a larger array of bucket lists if it turns out that there are thousands of symbols in this table. For the sake of argument, let's suppose that we've decided to go with a fixed-number-of-buckets hashing approach, as we've done here.

One of the much-touted benefits of immutable data structures is that they are "threadsafe". Since they cannot be written to, you'll never run into a situation where a write operation is interrupted halfway by a thread switch, causing another thread to read an inconsistent data structure. However, **it is a fallacy to believe that just because a data structure does not admit any way for *you* change its contents, its implementation must be threadsafe\!**

Suppose for example we do a real-world performance analysis of our little table here and discover that in practice, this sort of pattern comes up a lot in our customers' code:

 

    x.Foo();  
    x.Bar();  
    x.Blah();  
    y.Abc();  
    y.Def();  
    y.Ghi();

That is, the same identifier is looked up multiple times in a row in the context of a particular symbol table. We might decide to add a little optimization to our table's lookup method:

 

        for (var node = buckets\[bucket\]; node \!= null; node = node.Next)  
        {  
            if (node.Symbol.Name == name)  
            {  
                MoveToFront(bucket, node);  
                return node.Symbol;  
            }  
        }  
...  
    private void MoveToFront(int bucket, BucketListNode node)  
    {  
        if (buckets\[bucket\] == node)  
            return;  
        node.Prev.Next = node.Next;  
        if (node.Next \!= null)  
            node.Next.Prev = node.Prev;  
        node.Prev = null;  
        node.Next = buckets\[bucket\];  
        node.Next.Prev = node;  
        buckets\[bucket\] = node;  
    }

We have essentially tweaked our hash table to have an array of **Most Recently Used Lists**, rather than merely an array of **Doubly Linked Lists**. That might be a performance win in the case where the lists get long and the same identifiers tend to be looked up in clusters. (It is enough of a real-world win that in the VBScript engine, the lookup tables use a variant of this optimization.)

But you see what we've done here? Reading the table now sometimes mutates its interior structure, and that mutation is not threadsafe\! Even though there is no way for the user to logically change a frozen table, we do not ensure that reading the data is threadsafe.

In practice, most data structures do not do mutations on reads, but you cannot rely upon that unless it is documented. For example, the [documentation](http://msdn.microsoft.com/en-us/library/xfhwa508.aspx) for the Dictionary class notes that a Dictionary is threadsafe for multiple readers so long as there are no writers (though of course the actual *enumerators* are not threadsafe objects, as they are constantly mutating; rather, the collection *being enumerated* is threadsafe). Unless the author of a particular object makes a guarantee to you like that, you should assume that none of the operations are threadsafe, even if they appear to be read-only.

