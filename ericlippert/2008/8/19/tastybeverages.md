# Tasty Beverages

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/19/2008 5:09:00 PM

-----

“*Diet Dr. Pepper tastes more like regular Dr. Pepper.”*

That was a previous advertising slogan for Diet Dr. Pepper, my personal favourite source of both caffeine and phenylalanine; I’m drinking it right now as I write this.

The present slogan is the brain-achingly oxymoronic “*Diet Dr. Pepper: There’s Nothing Diet About It*” – really?  Seems like one ought to change the name then, if the name of one’s product is so misleading as to *require its complete and utter disavowal in the slogan*.

But that’s not what I want to talk about today. I actually want to talk about *predicates*.

The word “predicate” is one of those slippery words that has multiple technical meanings depending on the domain, all related but subtly different enough that one really ought to carefully call out how one is using the term.

  - In *C\# programming* a predicate is a function which takes a bunch of arguments and returns a Boolean. customer=\>customer.Age \< 21, for example.
  - In *mathematical logic*, a predicate is a function (or relation) which indicates the membership of a set. For example, “all integers x such that x is both even and prime” is a predicate for the set containing only the number 2.
  - In *English* *grammar*, a predicate is that part of a statement which claims to represent a fact about the subject. For example, in “*Thucydides fought in the Peloponnesian War*”, “[Thucydides](http://blogs.msdn.com/ericlippert/archive/2004/08/18/the-attribute-of-manliness.aspx)” is the subject and the rest is the predicate. This predicate happens to be true for the subject “Thucydides”, and false for a different subject, say, “Julius Caesar”.

What all of these things have in common is that **a predicate is something into which you can "substitute" a value to obtain a result of either truth or falsity.**

What on earth does this have to do with Diet Dr. Pepper tasting more like regular Dr. Pepper?

Is “*tastes more like regular Dr. Pepper*” actually a predicate? If it is then when it is applied to a subject it must produce a statement which can be classified as true or false. Let’s leave the subjective nature of taste aside for a moment; that’s not the fundamental logical problem here.

Rather, consider this utterance: “*Diet Dr. Pepper tastes more like*”  Is “*tastes more like*” a predicate? Of course not. That utterance doesn’t make any sense. Tastes more like… what? This utterance **cannot be classified as true or false for any subject**, so the latter part of it cannot actually be a predicate.

But the same thing goes for “*tastes more like regular Dr. Pepper*”\!  Tastes more like regular Dr. Pepper… than what?

In order to actually be a predicate it needs more objects. For example, “*Diet Dr. Pepper tastes more like regular Dr. Pepper than a pint of Guinness tastes like a mango lassi*” is a statement which actually has a truth value. Perhaps a subjective and arguable truth value, but at least this sentence has the form of a statement with a subject and a real predicate now. The original slogan’s “predicate” isn’t really a predicate at all; one might think of it as a **pseudopredicate**.

Advertisers *love* pseudopredicates. Once you realize that they exist you see them all the time. Advertisers love them because they make no testable claim which could be shown to be false in a court of law. Rather, they rely on either the irrational belief that “more” anything means “better”, or upon your brain’s ability to fill in the rest of the objects which they intend you to infer.

In this particular case, I imagine that the crafters of this slogan intended your brain to fill in “*Diet Dr. Pepper tastes more like regular Dr. Pepper than our previous formulation of Diet Dr. Pepper tasted like regular Dr. Pepper*” – that is, they want to make the claim that the product has improved without making the admission that the previous formulation was less than delicious.

Or, perhaps they want you to fill in “*Diet Dr. Pepper tastes more like regular Dr. Pepper than Diet Coke tastes like regular Coke*” – that is, they want to assert that their product is superior to a competing product. This assertion is in my personal opinion true, but the Coca Cola company could potentially take issue with if stated baldly as an objective claim. By relying upon a pseudopredicate to make a slogan which actually is so malformed as to have no truth value at all, the copywriters duck these thorny issues. There are lots of ways that clever advertisers leverage our tendancies to "fill in the blanks" in order to sell products.

That’s not actually what I want to talk about today either.  I *actually* want to talk about *writing secure code*.

What on earth does Diet Dr. Pepper tasting more like regular Dr. Pepper have to do with writing secure code?

The other day I got a question about the characteristics of a particular bit of source code obfuscation technology, which we shall call X; what the technology actually consists of and what the precise question was are irrelevant to this discussion.

I answered the question with a question; I asked why it was that the questioner wanted to use technology X. The answer was “*To protect the source code*”. Leave aside for the moment the fact that I could probably have deduced from the original query that the questioner was interested in protecting some resource. There’s a deeper problem here. In the utterance “*technology X protects the source code*”, is “*protects the source code*” a predicate, or a pseudopredicate?

It’s a pseudopredicate. There is an object missing. To make this a predicate, it needs to be something like “*protects the source code from casual inspection and editing by snoopy people*” – as it happens, this predicate was true for technology X. What I was rather worried about was that the questioner actually had in his head the predicate “*protects the database administrator password that I’ve stuck into my source code from discovery and misuse by a determined and intelligent attacker*”. That predicate happens to be utterly false for technology X. Because he was not actually stating a predicate that could be true or false of X I was unable to answer the guy's question about X.

I never, ever lock my car doors anymore. Why? I drive a soft-top convertible. One day I woke up to discover that someone had sliced open the top and unlocked the car. The two bucks in quarters I keep in the car for parking meters was a trivial loss compared to the hundreds my insurance company paid to get the top replaced and the hours of my time wasted in dealing with the situation. The locks are not a mitigation to that vulnerability at all\! Locking my car doors makes it *more* prone to be damaged, not less.

I do, however, lock my house, to protect it against random people wandering in. However, the locks are hardly any mitigation to the vulnerability of the house to determined attack from a wily, hostile burglar. It would be foolish of me to say that “*the locks protect my house*” without mentioning the **threat**.

What I’m rambling on about here is this: the fitness of a particular security technology to **mitigate a vulnerability** can only be evaluated in the context of a **stated threat** against a **stated resource**. That’s because **every security technology is designed to mitigate specific vulnerabilities to particular threats**. When you’re evaluating the benefit of a particular security system, make sure that the predicates you are using to talk about the system are actually predicates, not pseudopredicates; **state the threats**.

