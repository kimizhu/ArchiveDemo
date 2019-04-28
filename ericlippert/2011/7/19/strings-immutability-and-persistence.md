<div id="page">

# Strings, immutability and persistence

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/19/2011 2:16:37 PM

-----

<div id="content">

<div class="mine">

Todays post is [based on a question from StackOverflow](http://stackoverflow.com/questions/6742923/if-strings-are-immutable-in-net-then-why-does-substring-take-on-time); I liked it so much I figured hey, let's just blog it today.

When you look at a string in C\#, it looks to you like a collection of characters, end of story. But of course, behind the scenes there is a data structure in memory somewhere to implement that collection of characters. In the .NET CLR, [strings are laid out in memory pretty much the same way that BSTRs were implemented in OLE Automation](http://blogs.msdn.com/b/ericlippert/archive/2003/09/12/52976.aspx): as a word-aligned memory buffer consisting of a four-byte integer giving the length of the string, followed by the characters of the string in two-byte chunks of UTF-16 data, followed by two zero bytes. (Recall that BSTR originally stood for "BASIC string", because the OLE Automation team was actually part of the Visual Basic team; this is the format that Visual Basic used.)

Using this as the internal implementation of strings has a number of benefits. For example: it only requires one heap allocation per string. The length can be determined without counting characters. The string can contain embedded zero bytes, unlike formats that use zero bytes as end-of-string markers. If you disregard surrogate pairs then the nth individual character can be fetched in O(1) time, unlike in variable-width encodings like UTF-8. If the string is pinned in place and contains no zero characters then the address of the string data can be passed unmodified to unmanaged code that takes a WCHAR\*. And so on.

Strings are immutable in .NET, which also has many benefits. [As I've discussed many times, immutable data types are easier to reason about, are threadsafe, and are more secure](http://blogs.msdn.com/b/ericlippert/archive/tags/immutability/). (\*)

One of the benefits of the immutable data types I've talked about here previously is that they are not just immutable, they are also "**persistent**". By "persistent", I mean an immutable data type such that common operations on that type (like adding a new item to a queue, or removing an item from a tree) can **re-use most or all of the memory of an existing data structure**. Since it is all immutable, you can re-use its parts without worrying about them changing on you.

Strings in this format are immutable, but they are not persistent, thanks to that pesky single-buffer-with-both-a-prefix-and-suffix layout. Concatenation two strings does not re-use the content of either of the original strings; it creates a new string and copies the two strings into the new string. Taking the substring of a string does not re-use the content of the original string. Again, it just makes a new string of the right size and makes a full copy of the data.

This means that operations on strings such as taking a substring are O(n) in the size of the substring. Concatenations are O(n+m) in the sizes of the two source strings. These operations could be O(1) or O(lg n) if strings were persistent. For example, we could treat strings as [catenable deques](http://blogs.msdn.com/b/ericlippert/archive/2008/02/12/immutability-in-c-part-eleven-a-working-double-ended-queue.aspx) of chars; there are ways to do very efficient concatenations of catenable deques. (\*\*) Or, we could say that there are two kinds of strings; "regular" strings and "ropes", which are binary trees of strings. Concatenating two strings just allocates a new rope. It becomes a bit more expensive to find the nth char of a rope, and you can't pass them unmodified to managed code, but **you always avoid a potentially lengthy copy**. Similarly with substring operations: taking the substring of a string could simply allocate a wrapper around the original string that keeps an offset and a length. No copying required.

Why don't we do "persistent" optimizations like this, since we have an immutable data structure already?

The motivation for doing so is based on the incorrect idea that O(1) is always better than O(lg n), is always better than O(n). **The asymptotic order of an operation is only relevant if under realistic conditions the size of the problem actually becomes large**. If n stays small then every problem is O(1)\!

That is the situation we are actually in. Very few developers routinely deal with strings of more than a few hundred or a few thousand characters. If you are dealing with larger strings and doing a lot of concatenation, you can use a StringBuilder; if you're only doing a small number of concatenations, it is very fast to do the copy. If you're taking substrings, odds are very good that the substrings are going to be a few dozen characters out of a thousand character string, not a few hundred thousand characters out of a million-character string. Most line-of-business programmers are not in the business of chopping up strings containing million-character DNA strands; they're in the business of parsing the name of a book out of an XML document or an address out of a comma-separated-value text file. The memory operations for making a relatively small string buffer and copying a few dozen bytes out of it are insanely fast, so fast that there is really little point in optimizing it further.

**Unless you have really good reason to believe that the "optimization" is a win, it probably isn't.** I spent a summer about twelve years ago rewriting the VBScript internal string manipulations to use "ropes" -- to build up a tree on every string concatenation, and only turn it back into a "real" BSTR when necessary. Our research on real-world code showed that the "optimization" was no optimization at all; most of the time strings were being turned from a rope back into a BSTR after the second or third concatenation, and the added expense of allocating all the tree structures was actually *slower* than just making a copy of every BSTR every time. (\*\*\*)

Now, some people are actually manipulating relatively large strings and inserting and removing substrings on a regular basis. My team does that all the time; we write code editors that have to be able to deal with enormous files being rapidly edited. Because operations on strings are O(n) and n is actually big, we do not use big strings; rather, we have immutable persistent data structures that represent edits to the document, and those edits are then fed into an engine that maintains an immutable, persistent model of the lexical and syntactic analysis of the program. We want to be able to re-use as much of the previously-seen text and analysis as possible, to ensure high performance in both memory size and speed.

That was some tricky code to write, and what works for us almost certainly does not work for the people doing DNA strings, or any other big-string-with-lots-of-substrings problem that you care to name. Rather than optimize CLR strings for narrow, rare cases, it's better to just keep it simple, optimize for the common case, and rely on the hardware taking care of making blindingly fast copies of short strings.

\-------------

(\*) Consider an attack where partially-trusted hostile code passes a string containing a "safe" path to File.Open. The security checker verifies that the path is safe for partially-trusted code to open. And then the partially-trusted code mutates the string on another thread before the file is actually opened\! If they get the timing right then the security check passes and the wrong file is opened. If strings are immutable, this kind of thing does not happen.

(\*\*) My article on deques does not show how to make them efficiently catenable; see the referenced papers for details.

(\*\*\*) If you happen to be one of the people with access to the VBScript source code, I think all that gear is still in there, but turned off. Search for FANCY\_STRING in the sources.

</div>

</div>

</div>

