<div id="page">

# Reading Code Over the Telephone

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/16/2008 10:00:00 AM

-----

<div id="content">

<div class="mine">

In my youth I once attended a lecture given by Brian Kernighan on the subject of code quality, which was very influential on my attitudes towards writing legible code. One of the things that Kernighan recommended was to endeavour write code that was so clear that it could be easily read over the phone and understood. Most people find code much easier to comprehend when read than when heard; if you can make it clear enough to be understood when heard, it's probably pretty clear code.

I was reminded of this when I got the following question from a colleague by email:

<span class="code"> </span>

Subject: Stupid C\# 3.0 lambda expression question

How does one read the =\> operator?

First off, I told my colleague that **there are no stupid *questions*, only stupid *people***. No stupid people work here, so don't stress about it. This is a perfectly sensible question.

As far as I know, we do not have an "official" line on how to read this operator over the phone. In the absense of any other context, I personally would say <span class="code">c=\>c+1</span> as "see **goes to** see plus one". Some variations that I've heard:

For a projection, <span class="code">(Customer c)=\>c.Name</span>: "customer see **becomes** see dot name"

For a predicate, <span class="code">(Customer c)=\>c.Age \> 21</span>: "customer see **such that** see dot age is greater than twenty-one"

An unfortunate conflation is that the =\> operator looks a lot like ⇒, the "implies" operator in mathematics. Since =\> does not have the same semantics as ⇒, it is probably a bad idea to read =\> as "implies". (x⇒y would have the semantics of <span class="code">\!x|y</span> in C\#.)

Incidentally, it is a little known fact that VB6 and VBScript implemented the ⇒ operator with the <span class="code">Imp</span> keyword and the ⇔ operator with the <span class="code">Eqv</span> keyword. They disappeared in VB.NET. Where did they go? It is a mystery\!

</div>

</div>

</div>

