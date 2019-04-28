# What’s the difference between ternary and tertiary?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/18/2010 6:22:00 AM

-----

[![Tertiary](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Whatsthedifferencebetweenternaryandteria_C6C2/Tertiary_3.jpg "Tertiary")](http://swazzle.com/blogs/2006/02/color-theory-3.html) The conditional operator ( condition ? consequence : alternative ) is often referred to as both the “ternary operator” and the “tertiary operator”. What’s the difference?

“*Ternary*” means “*having three parts*”. Operators in C\# can be unary, binary or ternary – they take one, two or three operands.

 “*Tertiary*” means “*third in order*”. Compiler flaws noted in bug reports can be of primary, secondary or tertiary importance. Colours can be primary (yellow), secondary (orange) or [tertiary](http://swazzle.com/blogs/2006/02/color-theory-3.html) (yellowish-orange), like our muppet friend to the left there. And so on.

“Tertiary operator” is therefore an English usage error, unless what you’re trying to say is that the conditional operator is third most important to you, or that it is a lovely greenish-blue colour.

I say avoid the problem altogether; it is simply more clear to call the conditional operator “the conditional operator”.

