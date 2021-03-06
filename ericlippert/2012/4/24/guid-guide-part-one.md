<div id="page">

# GUID Guide, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/24/2012 6:32:00 AM

-----

<div id="content">

<div class="mine">

What is a GUID? The acronym stands for "globally unique identifier"; GUIDs are also called UUIDs, which stands for "universally unique identifier". (It is unclear to me why we need two nigh-identical names for the same thing, but there you have it.) A GUID is essentially a 128 bit integer, and when written in its human-readable form, is written in hexadecimal in the pattern {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}.

The purpose of a GUID is, as the name implies, to *uniquely identify* something, so that we can refer to that thing by its identifier and have confidence that everyone can agree upon what thing we are referring to. Think about this problem as it applies to, say, books. It is cumbersome to refer to a book by *quoting it in its entirety every time you mention it*. Instead, we give every book an identifier in the form of its title. The problem with using a title as an identifier is that there may be many different books with the same title. I have three different books all entitled "<span class="underline">The C\# Programming Language</span>" on my desk right now; if I want to refer to one of them in particular, I'd typically have to give the edition number. But there is nothing (apart from their good sense) stopping some entirely different publisher from also publishing a book called "The C\# Programming Language, fourth edition" that differs from the others.

Publishers have solved this problem by creating a globally unique identifier for each book called the [International Standard Book Number](http://en.wikipedia.org/wiki/ISBN), or ISBN. This is the 13-decimal-digit bar coded number you see on pretty much every book(\*). How do publishers manage to get a unique number for each of the millions of books published? They divide and conquer; the digits of an ISBN each have a different meaning. Each country has been assigned a certain range of ISBN numbers that they can allocate; governments then further allocate subsets of their numbers to publishers. Publishers then decide for themselves how to assign the remaining digits to each book. The ISBNs for my three editions of the C\# spec are 978-0-321-15491-6, 978-0-321-56299-9 and 978-0-321-74176-9. You'll notice that the first seven digits are exactly the same for each; they identify that this is a publishing industry code (978), that the book was published in a primarily English-speaking region (0), by Addison-Wesley (321). The next five digits are Addison-Wesley's choice, and the final digit is a checksum. If I wish to uniquely identify the fourth edition of the C\# specification I need not state the ambiguous title at all; I can simply refer you to book number 978-0-321-74176-9, and everyone in the world can determine precisely which book I'm talking about.

An important and easily overlooked characteristic of the ISBN uniqueness system is that **it only works if everyone who uses it is non-hostile**. If a rogue publisher decides to deliberately publish books with the ISBN numbers of existing books so as to create confusion then the usefulness of the identifier is compromised because it no longer uniquely identifies a book. ISBN numbers are not a *security system*, and neither are GUIDs; **ISBN numbers and GUIDs  prevent *accidental* collisions**. Similarly, traffic lights only prevent accidental collisions *if everyone agrees to follow the rules of traffic lights*; if anyone decides to go when the light is red then collisions might no longer be avoided, and if someone is attempting to deliberately cause a collision then traffic lights cannot stop them.

The ISBN system has the nice property that you can "decode" an ISBN and learn something about the book just from its number. But it has the enormous down side that it is *extraordinarily expensive to administer*. There has to be international agreement on the general form of the identifier and on what the industry and language codes mean. In any given country there must be some organization (either a government body or private companies contracted by the government) to assign numbers to publishers. It can cost hundreds of dollars to obtain a unique ISBN.

GUIDs do not have this cost problem; GUIDs are free and there is no requirement that any governing body get involved to ensure their uniqueness. A GUID is a number that you can generate yourself and be guaranteed that no one else in the world will generate that same number. That seems a bit magical. How does that work? Over the next couple of episodes we'll take a look at how that magical property is achieved.

-----

(\*) The attentive reader will note that there are usually *two* bar codes on a book in the United States. The first one is the ISBN; the second bar code is the number 5 followed by a four digit number that is the publisher's suggested price of the book in American pennies.

</div>

</div>

</div>

