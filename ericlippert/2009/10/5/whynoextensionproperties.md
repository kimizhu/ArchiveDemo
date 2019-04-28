# Why No Extension Properties?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/5/2009 9:29:00 AM

-----

I'm frequently asked "you guys added extension methods to C\# 3, so why not add extension properties as well?" 

Good question.

First, let me talk a bit about C\# 3. Clearly the big feature in C\# 3 was LINQ. In a sense we had only three features in C\# 3:

  - everything necessary for LINQ -- implicitly typed locals, anonymous types, lambda expressions, extension methods, object and collection initializers, query comprehensions, expression trees, improved method type inference
  - partial methods
  - automatically implemented properties

The latter two were tiny compared to the items in that first bucket. On the design side, the syntax and semantics of both features are straightforward. On the implementation side, we already had the mechanisms in place to remove a call site; partial methods and conditional methods are pretty much the same thing behind the scenes. Auto props were also straightforward to analyze and generate code for. The testing burden was not particularly large for these features either.

The C\# team was "the long pole" for the 2008 release of Visual Studio and the .NET Framework. By that I mean that if you took the amount of time required to do the work each team signed up for, given our level of staffing, blah blah blah, and made a pole proportionally long for each team in Developer Division, the C\# pole would have been the longest pole. Which means that every other team in devdiv had slack in their schedule, but if we slipped our schedule a day, the new VS/CLR would also ship a day late. If any other team slipped a day, well, as long as that didn't make their pole longer than ours, they were still OK. Within the team, dividing up the work amongs the various development team members, the "long pole" work was the lambda binding and method type inference work, which was mine. So in a sense, every day that I was late, we'd slip the whole product that many days. (No pressure\!)

Fortunately we have an excellent team here, we all supported each other very well through that release, picked up each other's slack when necessary, and delivered a quality release in plenty of time. My point is simply that **unless a feature was either necessary for LINQ, or small and orthogonal and easy to cut if necessary (like the other two), it got cut immediately**. There was no way we were going to risk slipping the entire product for any feature that was both *complex* and *unnecessary*.

It was of course immediately obvious that the natural companion to extension methods is extension properties. It's less obvious, for some reason, that extension events, extension operators, extension constructors (also known as "the factory pattern"), and so on, are also natural companions. But we didn't even *consider* designing extension properties for C\# 3; we knew that they were not necessary and would add risk to an already-risky schedule for no compelling gain.

So now we come to C\# 4.

As I'm fond of pointing out, the answer to every question of the form "why doesn't product X have feature Y?" is the same. It's because in order for a product to have a feature, that feature must be:

  - thought of in the first place
  - desired
  - designed 
  - specified
  - implemented
  - tested
  - documented
  - shipped to customers

You've got to hit every single one of those things, otherwise, no feature. 

When we started working on C\# 4, we made a list of every feature request we'd heard of. It had hundreds of features on it. [As I described last year](http://blogs.msdn.com/ericlippert/archive/2008/10/08/the-future-of-c-part-one.aspx), we categorized that list into "gotta have / nice to have / bad idea" buckets. Extension properties were in the "gotta have" bucket. We then looked at our available budget -- which was not so much measured in dollars as in available designers, developers, testers, writers and management multiplied by available time -- and determined that we did not have the resources to do more than about half the things in the "gotta have" bucket. So we cut half that stuff. Extension properties made it past that cut.

We then designed the feature. We had many hours of debate about the proposed syntax on the declaration side, how to call an extension property getter or setter "directly" as a static method, and so on. We came up with a syntax we could agree was acceptable, designed the semantics, wrote up a draft specification, and started writing code and test plans.

By the time the code was in reasonable shape -- not yet seriously tested, but usable and compliant with the spec -- we'd been having meetings with the WPF team. WPF developers were the assumptive primary consumers of extension properties. WPF already has a mechanism that resembles extension properties; it would be nice to unify that mechanism with our mechanism. Unfortunately, after taking a deep look at their real-world scenarios, we came to the disappointing conclusion that we had designed the wrong thing; this was not actually a feature that would solve their problems.

This is, admittedly, in a sense a failure of the design process. In retrospect, we should have gotten input and feedback from the primary customer much earlier. Had we done that then they could have either influenced the design to make something that would work for them, or we could have cut the feature without taking on the expense of writing spec, code and test plans.

But when you're in an unfortunate situation, you've got to decide to stop throwing good money after bad. Rather than take on the additional costs -- testing, documentation, shipping, and then maintenance of the feature for the rest of the life of the language -- in exchange for a feature that did not serve the needs of our customers, we cut the feature. At that point we did not feel confident that we had enough cycles remaining to redesign the feature the right way.

Therefore, sadly, no extension properties in C\# 4. Perhaps in a hypothetical future version of C\#.

