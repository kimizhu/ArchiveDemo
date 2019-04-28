# Should C\# warn on null dereference?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/17/2012 9:25:40 AM

-----

As you probably know, the C\# compiler does flow analysis on constants for the purposes of finding unreachable code. In this method the statement with the calls is known to be unreachable, and the compiler warns about it.

const object x = null;  
void Foo()  
{  
  if (x \!= null)  
  {  
    Console.WriteLine(x.GetHashCode());  
  }  
}

Now suppose we removed the "if" statement and just had the call.

const object x = null;  
void Foo()  
{  
  Console.WriteLine(x.GetHashCode());  
}

The compiler does *not* warn you that you're dereferencing null\! The question is, as usual, **why not?**

The answer is, as usual, that we do not have to provide a justification for not doing a feature. Features cost money, time and effort, and take away money, time and effort from features that would benefit the user better, so features have to be justified based on a cost-vs-benefits analysis. So let's think about that.

Something I find helpful when thinking about a particular feature is to ask "**is this a specific case of a more general feature?**" The proposed feature is essentially to detect that a particular exception is always going to be thrown. Do we want to in general be able to detect cases where an exception will always be thrown and warn about them? Well, doing so with certainty is equivalent to solving the Halting Problem, [as we've already discussed](http://blogs.msdn.com/b/ericlippert/archive/2011/02/24/never-say-never-part-two.aspx). But even without that, we do not want to give a warning every time we know that an exception must be thrown; that would then make this program fragment produce a warning:

int M()  
{  
  throw new NotImplementedException();  
}

The whole point of throwing that exception in the first place is to make it clear that this part of the program doesn't work; giving a warning saying that it doesn't work is counterproductive; warnings should tell you things you don't already know.

So let's just think about the feature of detecting when a constant null is dereferenced. How often does that happen, anyway? I occasionally make null constants; perhaps because I want to be able to say things like "if (symbol == InvalidSymbol)" where InvalidSymbol is the constant null; it makes the code read more like English. And maybe someone could accidentally say "InvalidSymbol.Name" and the compiler could warn them that they are dereferencing null.

We've been here before; in fact, [I made a list](http://blogs.msdn.com/b/ericlippert/archive/2011/03/03/danger-will-robinson.aspx). So let's go down it.

The feature is reasonable; the code seems plausible, it is clearly wrong, and we could detect that without too much expense. However, the scope of this warning is very small; the vast majority of the time I've used null constants, I've used them for comparison, not for accessing members off of them. And the problem will be detected when we test the code, every time.

Could we perhaps then generalize the feature in a different way? Perhaps instead of constants we should detect any time that the compiler can reasonably know that *any* dereference is probably a null dereference. Solving the problem perfectly is, again, equivalent to solving the Halting Problem, which we know is impossible, but we can use some clever heuristics to do a good job.

In fact the C\# compiler already has some of those heuristics; it uses them in its nullable arithmetic optimizer. If we can statically know that a nullable expression is always null or never null then we can do a better job of codegen. (And how we do so would be a great future blog topic.) However, the existing heuristics are extremely "local"; they do not, for example, notice that a local variable was assigned null and then later used before it was reassigned. So again, the scope of this warning would be very small, probably too small.

To do a good job of the proposed more general feature, we'd want to modify the existing flow analyzer that determines if a local variable is definitely assigned before it is used, with one that also can tell you whether the value assigned was non-null before a dereference. That's a much more expensive feature; the benefits are higher, but so are the costs.

What it really comes down to me for this feature is that last item on my list. Yes, it is always better to catch a bug at compile time, but that said, null dereference bugs *that the compiler could have told you about with certainty* are the easiest ones to catch at runtime *because they always happen the moment you test the code*. So that's some points against the feature.

So basically the feature request here is to write a somewhat expensive detector that detects at compile time a somewhat unlikely condition that will always be immediately found the first time the code is run anyways. It is therefore not a great candidate for spending budget on to implement it; thus the feature has never been implemented. **It's a lovely feature and I'd be happy to have it in the compiler, but it's just not big enough bang for the design, development and testing buck,** and we have many higher priorities.

Now, you might note that tools like the Code Contracts feature that shipped with version 4, or Resharper, or other similar tools, all have various abilities to statically detect possible or guaranteed null dereferences. Which proves that it is possible to do a good job of this feature, and that's good to know. But that is also points against doing the warning in the compiler. As I point out on my list, if an existing analysis tool does a good job, then why replicate that work in the compiler? The compiler does not aim to be the be-all and end-all of code analysis; in fact, we are building Roslyn precisely to make it easier for third parties to develop such analysis tools. We can't possibly do every great code analysis feature, but we can make it easier for you to do so\!

