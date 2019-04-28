# High maintenance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/8/2008 12:36:00 PM

-----

The other day I went to buy some snack from the snack machine in the kitchen. The snack I wanted was in slot B-10, so I put in my coins, press B - one - zero, hey wait a minute there's no zero button\! And why is it serving me up the snack on the left end of the machine instead of the right? Aha, there is a button marked "10", which is the one I was supposed to press. Instead I got snack B1. How irksome\!

And then I laughed at my plight, because of course Steve Maguire told *the same story* about Microsoft vending machines in Writing Solid Code *fifteen years ago*. Maguire went on to make an analogy between bad candy machine interfaces and bad software interfaces; a "candy machine interface" is one which leads the caller to make a plausible but wrong choice.

Coincidentally, I was asked to review a fragment of code recently that got me thinking again about candy machine interfaces. This isn't *exactly* the kind of bad interface that Maguire was talking about -- but I'm getting ahead of myself. Let's take a look at the code:

 

public static class StreamReaderExtensions  
{  
    public static IEnumerable\<string\> Lines(this StreamReader reader)  
    {  
        if (reader== null)  
            throw new ArgumentNullException("reader");  
        reader.BaseStream.Seek(0, SeekOrigin.Begin);  
        string line;  
        while ((line = reader.ReadLine()) \!= null)  
            yield return line;  
    }  
}

The *idea* of this code is awesome. I use this technique all the time in my own programs when I need to manipulate a file. Being able to treat a text file as a sequence of lines is very handy. However, the *execution* of it has some problems.

The first flaw is the bug I discussed in my earlier post on [psychic debugging](http://blogs.msdn.com/ericlippert/archive/2007/09/05/psychic-debugging-part-one.aspx). Namely, the null check is not performed when the method is called, but rather, when the returned enumerator is moved for the first time. That means that the exception isn't thrown until possibly far, far away from the actual site of the error, which is potentially confusing.

The second flaw is that the "while" loop condition is a bit hard to read because it tries to do so much in one line. Shorter code is not magically faster than longer code that does the same thing; write the code so that it is maximally readable.

But there's a deeper flaw here. To get at it, first let me state a crucial fact about the relationship between a stream reader and a stream:

A StreamReader “owns” its underlying stream. That is, once a stream has been handed to a stream reader by a caller, the caller should not muck with the stream ever again. The stream reader will be seeking around in the stream; mucking around with the stream could interfere with the stream reader, and the stream reader will interfere with anyone trying to use the stream. The stream reader emphasizes its ownership by *taking responsibility for disposing the stream when the stream reader is itself disposed.*

Now, “Lines” defines an object, and what does that object do right off the bat?  **It mucks around with the underlying stream.** This is big red flag \#1. This smells *terrible*. No one should be messing with that stream but the reader.

Furthermore, think about it from the caller’s point of view. Maybe the caller knows that there are a bunch of bytes that it wants to skip, so it deliberately hands a StreamReader to Lines() which has been positioned somewhere past the beginning of the file. But Lines() thinks that it knows best and **ignores the information that the caller has given it**. This is big red flag \#2.

(The reason why the original code was seeking back was because the same reader was being "recycled" many times to read the same file over and over again, which is yet another red flag. Readers are cheap; you can have multiple readers on one file, there's no need to hoard them and reuse them.)

The third big red flag for me here is the ownership issue. When you hand a stream to a reader, the reader takes care of everything for you from then on – that’s the “contract” between the reader and the caller.  “Lines()” does not have that contract. Lines()’s contract says *“attention caller: I am taking ownership of this reader. I am going to muck with its underlying stream. You must never use this reader for anything ever again – except that I’m not going to dispose it for you. If you want the reader and its stream closed then you are going to have to keep a reference to it around until **you** can prove that **I** am done with it. And if you get it wrong, either I crash or you get a resource leak. So get it right.”*

This is a *terrible* contract to impose upon a caller, and of course, the imposition here is in no way represented by the signature of the method. You just have to know that “Lines()” owns the reader for the purposes of all its *functionality*, but not for its *cleanup*, which is deeply weird.

In short, **the caller needs to know all the details of what is happening in the method in order to use it correctly**; this is a violation of the whole purpose of creating methods. Methods are supposed to abstract away an operation so that you do not have to know what they are doing.

A fourth red flag is that **the task performed by Lines() does not have a clear meaning in terms of the logic of the program**. In looking at every caller of this method it became clear that the desired semantics were “give me the lines of this **text file** one at a time”. But we haven’t written a method that does that. In order to get the lines of a text file from Lines() the caller is required to do all kinds of work to make Lines() happy – open the stream, create a reader, call Lines() but keep the reader around, close the reader after we know that the iterator is done.

This code makes the caller do a whole bunch of things -- things that have to be done in exactly the right order but potentially distributed out over long stretches of time and disparate code locations. This method is very "high-maintenance"\! Everything has to be just right all the time in order for it to be happy and get along with others; anything out of place and things will start going wrong.

Finally, we have an example of premature generality here -- if the intention is to read a text file, then write a method that reads a text file. If you don't need the power of reading lines from an arbitrary stream, maybe don't implement that. Cut down on your testing burden.

Some relatively simple changes fix it up:

 

public static class FileUtilities  
{  
    public static IEnumerable\<string\> Lines(string filename)  
    {  
        if (filename == null)  
            throw new ArgumentNullException("filename");  
        return LinesCore(filename);  
    }  
    private static IEnumerable\<string\> LinesCore(string filename)  
    {  
        Debug.Assert(filename \!= null);  
        using(var reader = new StreamReader(filename))  
        {  
            while (true)  
            {  
                string line = reader.ReadLine();  
                if (line == null)  
                   yield break;  
                yield return line;  
            }  
        }  
    }  
}

And now everything is crystal clear. The purpose of this thing is to read the lines out of a text file. The caller gives it the name of a file, it gives you the lines, and we’re done. The caller does not have to worry about anything else – the iterator takes care of opening the file, cleaning up when its done, and so on. There’s no messing around with streams at all; we have now provided **an abstraction over the file system** to the caller.

Whenever you write a method **think about the contract of that method**. What burdens are you imposing upon the caller? Are they reasonable burdens?  The purpose of a method should be to make the caller’s life easier; the original version of Lines() makes life harder on the caller. The new version makes life easier. **Don't write high-maintenance methods.**

