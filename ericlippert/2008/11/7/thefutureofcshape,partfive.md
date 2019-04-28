# The Future Of C\#, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/7/2008 11:18:00 AM

-----

When we're designing a new version of the language, something we think about a lot is whether the proposed new features "hang together". Having a consistent "theme" to a release makes it easier for us to prioritize and organize proposals, it makes it easier for our marketing and user education people to effectively communicate with customers, it's just all-around goodness.

If you look at C\# 2.0, it was a bit of a grab-bag. The big features were clustered around the notion of enabling rich, typesafe programming with abstract data types that represent collections of data -- and thus generic types and iterator blocks. But there was a whole lot of other stuff in there as well: implementing anonymous methods was a major feature that doesn't fit well with this theme. And there were other more minor features as well: partial classes, improvements to properties, and so on.

With C\# 3.0, the theme was very clear: **language-integrated query**. Anything that did not directly support LINQ was immediately made lower priority. It is rather amazing to me that partial methods and auto-implemented properties got in at all; that they were relatively easy features to design, implement, test and document was what saved them.

What then is the theme of C\# 4.0? Again, it seems like rather a grab-bag: covariance and contravariance, improved interop with dynamic languages, improved interop with legacy COM object models, named and optional parameters. It also seems like a pretty small set of new features compared to generics or query comprehensions.

That was deliberate. Some feedback that we received loud and clear throught the C\# 3.0 ship cycle was "*this* *is awesome, we need these language features immediately\!*" and, somewhat contradictorily, "*please stop fundamentally changing the way I think about programming every couple years\!*" Rather than trying to find some way to yet again radically increase the expressiveness and power of the language, we decided to spend a cycle on making what we already have work better with the other stuff in our programming platform infrastructure.

"*Now actually works the way you'd expect it to*" is not really a theme that gets people excited, but sometimes you've got to stop running forward at full speed and take some time to fix the existing stuff that is annoying a lot of people. (When I was on the VSTO team I petitioned the C\# team to please, please make ref parameters optional on calls to legacy COM object models, but they were too busy with designing LINQ; I'm delighted that we've finally gotten that in.)

We also want to make sure that we are anticipating the problems that people are about to face and mitigate them now. We know that dynamic languages and object models designed with dynamism in mind are becoming increasingly popular. Given that there will be stronger demand for statically typed C\# to interoperate with them in the future, let's get dynamic programming interoperability in there proactively, rather than be reactive about it later.

Looking forward, it's not clear what exactly the theme of future (hypothetical\!) versions of the language will be. The expected onslaught of cheap hardware parallelism looms large in our minds, so that's a possible theme. Enabling metaprogramming is another possible theme on our minds, thought it is not at all clear how that would happen. (Make C\# its own metalanguage? Extend expression trees to statement trees, declaration trees, and so on? Open up the internals of the compiler and provide an object model that lets people generate programs directly? It is hard to say what direction is the right one to go in here.) Fortunately, people way smarter than I am are thinking about these things.

