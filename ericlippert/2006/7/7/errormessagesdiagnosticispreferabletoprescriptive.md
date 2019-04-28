# Error messages: diagnostic is preferable to prescriptive

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/7/2006 2:04:00 PM

-----

The new LINQ features are going to create new failure modes for the compiler, so we're going to need to create some new error messages. The compiler development team got together the other day to discuss what makes an error message good or bad. I thought I'd share with you guys what we came up with. We believe that good error messages are:

  - **Polite:** making the user feel like an idiot is very, very bad.
  - **Readable:** poor grammar and tortured sentence structure is bad.
  - **Accurate:** error messages must accurately describe the problem.
  - **Precise:** "Something is wrong" is an *accurate* error message but not a very *precise* one\!
  - **Diagnostic** but not **prescriptive:** describe the *problem*, not the *solution*.

The first four are obviously goodness. That last one is a little more controversial. Surely a good error message not only tells you what is wrong but helps you fix it, no?

The issue is that deducing what is wrong with bad code is hard enough. Trying to read the user's mind and figure out what they were thinking when they wrote the bad code, and then telling them how to correctly implement that thought is not something that we feel we can do with sufficiently high accuracy in most situations.

Look at it this way: suppose we pull off a miracle and manage to produce error messages which 90% of the time tell the user the correct way to fix their code so that it does what they want it to do. That means that 10% of the time we are telling people how to write a syntactically correct program that does something different than they intended\! Pushing people towards writing buggy programs that still compile is very bad, and we do not want to go there.

It's instructive to look at a few places where we violated these guidelines in earlier versions of the compiler:

Static member 'Baz' cannot be marked as override, virtual or abstract 

If the user wrote static virtual, then we don't know what the heck they meant to do. Assuming that they meant to say static and that the virtual is wrong is a little presumptuous. Maybe the static is the wrong part\! Also, if the user said static virtual, then why is the error message mentioning override and abstract? That's accurate but not precise. A better error message in this case would be something like

Member 'Baz' cannot be both static and virtual 

Here's another place where we get it wrong, but this one is more subtle:

A params parameter must be the last parameter in a formal parameter list 

This is an example of an English sentence that can be interpreted different ways depending on the context. If I said "a punctuation mark must be the last symbol of a sentence" then I mean that *every* sentence must end in a punctuation mark, but I do not mean that punctuation marks are *only* legal at the end of a sentence. If I said "a period must be the last symbol of a statement" then I mean that *every* statement must end in a period, and furthermore that periods are forbidden anywhere else in the statement.

You and I know that what the error message is trying to say is that *if* there is a params then it must go at the end. But based solely on this error message, a user would be entirely logically justified in thinking that (int i) is an illegal parameter list because it doesn't end with a params parameter. Or, under another interpretation, they'd also be logically justified in concluding that (params int\[\] foo, params int\[\] bar) is legal, because it *does* end with a params parameter.

The portion of the specification which the error message is attempting to draw attention to is of course **"If a formal parameter list includes a parameter array then it must be the last parameter in the list. There can only be one parameter array for a given method"** which is nicely unambiguous. Why not simply use this quote from the specification for the error message? That's a reasonable idea, but it sounds a little stiff and doesn't call out where the problem is. I'd prefer:

Method 'Foo' has a parameter array parameter which is not the last parameter in the formal parameter list. 

This tells you what is wrong without telling you how to fix it. Since we don't know how to fix it – whether the user should be removing the params modifier, or moving it to the end, or rewriting their method from scratch – we should just report the spec violation and let them sort it out.

There *are* times when we do want to tell the user what to do, but only when we are *highly* likely to be correct. For example:

User-defined operator 'Blah' must be declared static and public. 

Here we are both diagnosing the problem and prescribing a solution. If they're trying to make a user-defined operator, this is what they absolutely must do to be successful. It is very unlikely that they wanted to make a private instance *function* and made a private instance *operator* by mistake\!

This illustrates another principle of good error messages that I didn't call out before: good error messages **use precise terminology from the standard** rather than making up new jargon. Yes "formal parameter list" and "user-defined operator" are a little bit stiff, but they are also clearly defined in the standard.

Sometimes we get the error right but the wording could be improved:

Foo: static classes cannot be used as constraints 

Why are they trying to use a static class as a constraint? Who knows? How should they fix it? Beats me\! The best we can do is to tell them that it hurts when they try to do that. But the wording\! Good heavens\! Would you ever say "Pizza: delicious foods should be eaten while they're fresh\!" ??? Clearly

Static class 'Foo' cannot be used as a constraint 

is much better.

Anyone have additional suggestions for what makes a good error message? Or other examples of places where we got it wrong?

