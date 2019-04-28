# Zip Me Up

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/7/2009 10:19:00 AM

-----

Suppose you’ve got a sequence of Foos and you want to project from that a sequences of Bars. That’s straightforward using LINQ:

 

IEnumerable\<Bars\> bars = from foo in foos select MakeBar(foo);

or, without the query sugar:

 

IEnumerable\<Bars\> bars = foos.Select(foo=\>MakeBar(foo));

But what if you have two sequences that you want to project from? Say you’ve got two sequences of doubles that are the same length, and you want to project a sequences of points.

The operation of “project from two sequences of the same length” is called a “zip join”, because its like doing up a zipper – you need each member of the left sequence to correspond to a member of the right sequence. In the version of the base class library that will ship with C\# 4.0, we’ll be adding a Zip sequence operator to the standard extension methods.

The code to do so is pretty trivial; if you happen to need this in C\# 3.0, I’ve put the source code below.

We *considered* adding a query syntax for zip joins. Something like

 

from foo in foos, bar in bars select MakeBlah(foo, bar)

or

 

from (foo, bar) in (foos, bars) select MakeBlah(foo, bar)

or some such thing, which would be syntactically transformed into

 

foos.Zip(bars, (foo, bar)=\>MakeBlah(foo, bar))

However, we decided that adding a query syntax for a relatively obscure operation that is not actually supported by most relational databases was not worth the cost of complicating the grammar and the syntactic translation rules.

Here’s the source code:

 

public static IEnumerable\<TResult\> Zip\<TFirst, TSecond, TResult\>  
    (this IEnumerable\<TFirst\> first,  
    IEnumerable\<TSecond\> second,  
    Func\<TFirst, TSecond, TResult\> resultSelector)  
{  
    if (first == null) throw new ArgumentNullException("first");  
    if (second == null) throw new ArgumentNullException("second");  
    if (resultSelector == null) throw new ArgumentNullException("resultSelector");  
    return ZipIterator(first, second, resultSelector);  
}

private static IEnumerable\<TResult\> ZipIterator\<TFirst, TSecond, TResult\>  
    (IEnumerable\<TFirst\> first,  
    IEnumerable\<TSecond\> second,  
    Func\<TFirst, TSecond, TResult\> resultSelector)  
{  
    using (IEnumerator\<TFirst\> e1 = first.GetEnumerator())  
        using (IEnumerator\<TSecond\> e2 = second.GetEnumerator())  
            while (e1.MoveNext() && e2.MoveNext())  
                yield return resultSelector(e1.Current, e2.Current);  
}

