# What curious property does this string have?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/12/2011 12:19:00 PM

-----

There are all kinds of interesting things in the Unicode standard. For example, the block of characters from U+A000 to U+A48F is for representing syllables in the "Yi script". Apparently it is [a Chinese language writing system](http://en.wikipedia.org/wiki/Yi_script) developed during the Tang Dynasty.

A string drawn from this block has an unusual property; the string consists of just two characters, both the same: a repetition of character U+A0A2:

string s = "ꂢꂢ";

Or, if your browser can't hack the Yi script, that's the equivalent of the C\# program fragment:

string s = "\\uA0A2\\uA0A2";

**What curious property does this string have?**

I'll leave some hints in the comments, and post the answer next time..

UPDATE: A couple people have figured it out, so don't read the comments too far if you don't want to be spoiled. I'll post a[follow-up article on Friday](http://blogs.msdn.com/b/ericlippert/archive/2011/07/15/the-curious-property-revealed.aspx).

