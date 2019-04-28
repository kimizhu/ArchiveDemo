# Precision and accuracy of DateTime

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/8/2010 6:37:00 AM

-----

[![Stopwatch](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/PrecisionandaccuracyofDateTime_A235/Stopwatch_thumb.jpg "Stopwatch")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/PrecisionandaccuracyofDateTime_A235/Stopwatch_2.jpg)The [DateTime](http://msdn.microsoft.com/en-us/library/system.datetime.aspx) struct represents dates as a 64 bit number that measures the number of “ticks” since a particular start date. Ten million ticks equals one second.

That’s a quite high degree of *precision*. You can represent dates and times to sub-microsecond accuracy with a DateTime, which is typically more precision than you need. Not always, of course; on modern hardware you can probably execute a couple hundred instructions in one tick, and therefore if you want timings that are at the level of precision needed to talk about individual instructions, the tick is too coarse a measure.

The problem that arises with having that much precision is of course that it is very easy to assume that a given value is as *accurate* as it is *precise*. But that’s not warranted at all\! I can represent my height in a double-precision floating point number as 1.799992352094 metres; though *precise* to a trillionth of a metre, it’s only *accurate* to about a hundredth of a metre because I do not have a device which can actually measure my height to a trillionth of a meter, or even a thousandth of a metre. There is way more precision than accuracy here.

The same goes for dates and times. Your DateTime might have *precision* down to the sub-microsecond level, but does it have *accuracy*? I synchronize my computers with time.gov fairly regulary. But if I don’t do so, their clocks wander by a couple of seconds a year typically. Suppose my clock loses one second a year. There are 31.5 million seconds in a year and 10 million ticks in a second, so therefore it is losing one tick every 3.15 seconds. Even if my clock was miraculously accurate down to the level of a tick at some point, within ten seconds, it’s already well off. **Within a day much of the precision will be garbage.**

If you do a little experiment you’ll see that the operating system actually gives you *thousands of times less accuracy than precision* when asked “what time is it?”

 

long ticks = DateTime.Now.Ticks;  
while(true)  
{  
    if (ticks \!= DateTime.Now.Ticks)  
    {  
        ticks = DateTime.Now.Ticks;  
        Console.WriteLine(ticks);  
    }  
    else  
    {  
        Console.WriteLine("same");  
    }  
}

On my machine this says “same” eight or nine times, and then suddenly the Ticks property jumps by about 160000, which is 16 milliseconds, a 64th of a second. (Different flavours of Windows might give you different results, depending on details of their thread timing algorithms and other implementation details.)

As you can see, the clock *appears* to be precise to the sub-microsecond level but it is *in practice* only precise to 16 milliseconds. (And of course whether it is *accurate* to that level depends on how accurately the clock is synchronized to the official time signal.)

Is this a flaw in DateTime.Now? Not really. The purpose of the “wall clock” timer is to produce dates and times for typical real-world uses, like “what time does Doctor Who start?” or “when do we change to daylight savings time?” or “show me the documents I edited last Thursday after lunch.”  These are not operations that require submicrosecond accuracy.

(And incidentally, in VBScript the “wall clock” timer methods built in to the language actually round off times we get from the operating system to the nearest second, not the nearest 64th of a second.)

In short, the question “what time is it?” really should only be answered to a level of precision that reflects the level of accuracy inherent in the system. Most computer clocks are not accurately synchronized to even within a millisecond of official time, and therefore precision beyond that level of accuracy is a lie. It is rather unfortunate, in my opinion, that the DateTime structure does surface as much precision as it does, because it makes it seem like operations on that structure ought to be accurate to that level too. But they almost certainly are not that accurate.

Now, the question “how much time has elapsed from start to finish?” is a completely different question than “what time is it right now?” If the question you want to ask is about how long some operation took, and you want a high-precision, high-accuracy answer, then use the [StopWatch](http://msdn.microsoft.com/en-us/library/system.diagnostics.stopwatch.aspx) class. It really does have nanosecond precision and accuracy that is close to its precision.

Remember, you don’t *need* to know *what time it is* to know *how much time has elapsed*. Those can be two different things entirely.

