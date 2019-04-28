<div id="page">

# Covariance and Contravariance in C\#, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/16/2007 11:04:00 AM

-----

<div id="content">

<div class="mine">

I have been wanting for a long time to do a series of articles about covariance and contravariance (which I will shorten to “variance” for the rest of this series.)

I’ll start by defining some terms, then describe what variance features C\# 2.0 and 3.0 already support today, and then discuss some ideas we are thinking about for hypothetical nonexistant future versions of C\#.

As always, keep in mind that we have not even shipped C\# 3.0 yet. **Any of my musings on possible future additions to the language should be treated as playful hypotheses, rather than announcements of a commitment to ship any product with any feature whatsoever.**

Today: what do we mean by “covariance” and “contravariance”?

The first thing to understand is that for any two types <span class="code">T</span> and <span class="code">U</span>, exactly one of the following statements is true:

  - <span class="code">T</span> is bigger than <span class="code">U</span>.
  - <span class="code">T</span> is smaller than <span class="code">U</span>.
  - <span class="code">T</span> is equal to <span class="code">U</span>.
  - <span class="code">T</span> is not related to <span class="code">U</span>.

For example, consider a type hierarchy consisting of <span class="code">Animal</span>, <span class="code">Mammal</span>, <span class="code">Reptile</span>, <span class="code">Giraffe</span>, <span class="code">Tiger</span>, <span class="code">Snake</span> and <span class="code">Turtle</span>, with the obvious relationships. ( <span class="code">Mammal</span> is a subclass of <span class="code">Animal</span>, etc.) <span class="code">Mammal</span> is a bigger type than <span class="code">Giraffe</span> and smaller than <span class="code">Animal</span>, and obviously equal to <span class="code">Mammal</span>. But <span class="code">Mammal</span> is neither bigger than, smaller than, nor equal to <span class="code">Reptile</span>, it’s just different.

Why is this relevant? Suppose you have a variable, that is, a storage location. Storage locations in C\# all have a type associated with them. **At runtime you can store an object which is an instance of an equal or smaller type in that storage location.** That is, a variable of type <span class="code">Mammal</span> can have an instance of <span class="code">Giraffe</span> stored in it, but not a <span class="code">Turtle</span>.

This idea of storing an object in a typed location is a specific example of a more general principle called the “substitution principle”. That is, in many contexts you can often substitute an instance of a “smaller” type for a “larger” type.

Now we can talk about variance. Consider an “operation” which manipulates types. If the results of the operation applied to any <span class="code">T</span> and <span class="code">U</span> always results in two types <span class="code">T’</span> and <span class="code">U’</span> with the *same* relationship as <span class="code">T</span> and <span class="code">U</span>, then the operation is said to be “covariant”. If the operation *reverses bigness and smallness* on its results but *keeps equality and unrelatedness the same* then the operation is said to be “contravariant”.

That’s totally highfalutin and probably not very clear. Next time we’ll look at how C\# 3 implements variance at present.

</div>

</div>

</div>

