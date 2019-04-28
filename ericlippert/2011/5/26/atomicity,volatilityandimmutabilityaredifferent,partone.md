# Atomicity, volatility and immutability are different, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/26/2011 8:07:00 AM

-----

I get a fair number of questions about atomicity, volatility, thread safety, immutability and the like; the questions illustrate a lot of confusion on these topics. Let's take a step back and examine each of these ideas to see what the differences are between them.

First off, what do we mean by "atomic"? From the Greek ἄτομος, meaning "not divisible into smaller parts", an "atomic" operation is one which is always observed to be done or not done, but never halfway done. The C\# specification clearly defines what operations are "atomic" in section 5.5. The atomic operations are reads and writes of variables of any reference type, or, effectively, any built-in value type that takes up four bytes or less, like int, short and so on. Reads and writes of variables of value types that take more than four bytes, like double, long and decimal, are not guaranteed to be atomic by the C\# language.

What does it mean for a read and write of an int to be atomic?  Suppose you have static variables of type int. X is 2, Y is 1, Z is 0. Then on one thread we say:

 

Z = X;

and on another thread:

 

X = Y

Each thread does one read and one write. Each read and write is itself atomic. What is the value of Z? Without any synchronization, the threads will race. If the first thread wins then Z will be 2. If the second thread wins then Z will be 1. But Z will definitely be one of those two values, you just can't say which.

David Corbin asks in a comment to my previous entry whether immutable structs are guaranteed to be written atomically regardless of their size. The short answer is no; why would they be? Consider:

 

struct MyLong  
{  
    public readonly int low;  
    public readonly int high;  
    public MyLong(low, high)  
    {  
        this.low = low;  
        this.high = high;  
    }  
}

Ignore for the moment the evil that is public fields. Suppose we have a fields Q, R and S of type MyLong initialized to (0x01234567, 0x0BADF00D), (0x0DEDBEEF, 0x0B0B0B0B) and (0, 0),  respectively. On two threads we say:

 

S = Q;

and

 

Q = R;

We have two threads. Each thread does one read and one write, but the reads and writes are not atomic. They can be divided\! This program is actually the same as if the two threads were:

 

S.low = Q.low;  
S.high = Q.high;

and

 

Q.low = R.low;  
Q.high = R.high;

Now, **you** can't do this because that's writing to a readonly field outside a constructor. But the CLR is the one enforcing that rule; it can break it\! (We'll come back to this in the next episode; things are even weirder than you might think.) Value types are copied by value; that's why they're called value types. When copying a value type, the CLR doesn't call a constructor, it just moves the bytes over one atomic chunk at a time. In practice, maybe the jitter has special registers available that allow it to move bigger chunks around, but that's not a guarantee of the C\# language. The C\# language only goes so far as to guarantee that the chunks are not smaller than four bytes.

Now the threads can race such that perhaps first S.low = Q.low runs, then Q.low = R.low runs, then Q.high = R.high runs, and then S.high = Q.high runs, and hey, S is now (0x0DEDBEEF, 0x0BADF00D), even though that was neither of the original values. The values have been splinched, as Hermione Granger would say (were she a computer programmer).

(And of course, the ordering above is not guaranteed either. The CLR is permitted to copy the chunks over in any order it chooses; it could be copying high before low, for example.)

The name "MyLong" was of course no accident; in effect, a two-int readonly struct is how longs are implemented on 32 bit chips. Each operation on the long is done in two parts, on each 32 bit chunk. The same goes for doubles, the same goes for anything larger than 32 bits. If you try reading and writing longs or doubles in multiple threads on 32 bit operating systems without adding some sort of locking around it to make the operation atomic, your data are highly likely to get splinched.

The only operations that are guaranteed by the C\# language to be atomic without some sort of help from a lock or other synchronization magic are those listed above: individual reads and writes of variables of the right size. In particular, operations like "increment" and "decrement" are not atomic. When you say

 

[i++;](http://blogs.msdn.com/b/ericlippert/archive/2007/12/13/immutability-in-c-part-five-lolz.aspx)

that's a syntactic sugar for "read i, increment the read value, write the incremented value back to i". The read and write operations are guaranteed to be atomic, but the entire operation is not; it consists of multiple atomic operations and therefore is not itself atomic. Two attempts to increment i on two different threads could interleave such that one of the increments is "lost".

There are many techniques for making non-atomic operations into atomic operations; the easiest is simply to wrap every access to the variable in question with a lock, so that it is never the case that two threads are messing with the variable at the same time. You can also use the [Interlocked](http://msdn.microsoft.com/en-us/library/system.threading.interlocked.aspx) family of helper methods which provide atomic increment, atomic compare-and-exchange, and so on.

Have a lovely Memorial Day weekend, American readers. I'm spending my Memorial Day weekend marrying a close personal friend(\*). Should be fun\!

**Next time:** readonly inside a struct is the moral equivalent of cheque kiting, plus ways you can make the atomicity guarantees stronger or weaker.

(\*) Actually, I am marrying \*two\* close personal friends. To each other, even\!

