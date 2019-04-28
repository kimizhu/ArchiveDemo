# Why does char convert implicitly to ushort but not vice versa?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/1/2009 12:53:00 PM

-----

[Another good question from StackOverflow](http://stackoverflow.com/questions/1503430/implicit-type-cast-in-c/1504959#1504959). Why is there an implicit conversion from char to ushort, but only an explicit conversion from ushort to char? Why did the designers of the language believe that these asymmetrical rules were sensible rules to add to the language?

Well, first off, the obvious things which would *prevent* either conversion from being implicit do not apply. A char is implemented as an unsigned 16 bit integer that represents a character in a UTF-16 encoding, so it can be converted to or from a ushort without loss of precision, or, for that matter, without change of representation. The runtime simply goes from treating this bit pattern as a char to treating the same bit pattern as a ushort, or vice versa.

It is therefore *possible* to allow either implicit conversion. Now, just because something is possible does not mean it is a good idea. Clearly the designers of the language thought that implicitly converting char to ushort was a good idea, but implicitly converting ushort to char is not. (And since char to ushort is a good idea, it seems reasonable that char-to-anything-that-ushort-goes-to is also reasonable, hence, char to int is also good.)

Unlike you guys, I have the original notes from the language design team at my disposal. Digging through those, we discover some interesting facts.

The conversion from ushort to char is covered in the notes from April 14th, 1999, where the question of whether it should be legal to convert from byte to char arises. **In the original pre-release version of C\#, this was legal for a brief time.** I've lightly edited the notes to make them clear without an understanding of 1999-era pre-release Microsoft code names. I've also added emphasis on important points:

> \[The language design committee\] has chosen to provide an implicit conversion from bytes to chars, since the domain of one is completely contained by the other. Right now, however, \[the runtime library authors\] only provide Write methods which take chars and ints, which means that bytes print out as characters since that ends up being the best method. We can solve this either by providing more methods on the writer class or by removing the implicit conversion.
> 
> There is an argument for why the latter is the correct thing to do. After all, **bytes really aren't characters**. True, there may be a *useful mapping* from bytes to chars, but ultimately, **23 does not denote the same thing as the character with ASCII value 23, in the same way that the byte 23 denotes the same thing as the long 23**. Asking \[the library authors\] to provide this additional method simply because of how a quirk in our type system works out seems rather weak.

The notes then conclude with the decision that byte-to-char should be an explicit conversion, and integer-literal-in-range-of-char should also be an explicit conversion.

Note that the language design notes do not call out why ushort-to-char was also made explicit at the same time, but you can see that the same logic applies. When passing a ushort to a method overloaded as M(int) and M(char), odds are good that you want to treat the ushort as a number, not as a character. And a ushort is not a character representation in the same way that a ushort is a numeric representation, so it seems reasonable to make that conversion explicit as well.

The decision to make char go to ushort implicitly was made on the 17th of September, 1999; the design notes from that day on this topic simply state "char to ushort is also a legal implicit conversion", and that's it. No further exposition of what was going on in the language designers' heads that day is evident in the notes.

However, we can make *educated guesses* as to why implicit char-to-ushort was considered a good idea. The key idea here is that the conversion from number to character is a "possibly dodgy" conversion. It's taking something that you do not know is intended to be a character, and choosing to treat it as one. That seems like the sort of thing you want to call out that you are doing explicitly, rather than accidentally allowing it. But the reverse is much less dodgy. There is a long tradition in C programming of treating characters as integers -- to obtain their underlying values, or to do mathematics on them.

In short: it seems reasonable that using a number as a character could be an accident and a bug, but it also seems reasonable that using a character as a number is deliberate and desirable. This asymmetry is therefore reflected in the rules of the language.

