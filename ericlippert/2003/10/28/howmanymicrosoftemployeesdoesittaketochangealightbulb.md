# How many Microsoft employees does it take to change a lightbulb?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/28/2003 4:41:00 PM

-----

UPDATE: This article was featured in [The Best Software Writing I](http://www.amazon.com/gp/product/1590595009/002-4836772-4674411?v=glance&n=283155). Thanks Joel\!

Joe Bork has written a [great article](http://headblender.com/joe/blog/archives/microsoft/001280.html)explaining some of the decisions that go into whether a bug is fixed or not. This means that I can cross that one off my list of potential future entries. Thanks Joe\!

But while I'm at it, I'd like to expand a little on what Joe said.His comments generalize to more than just bug fixes. **A bug fix is one kind of change to the behaviour of the product, and all changes have similar costs and go through a similar process.**

Back when I was actually adding features to the script engines on a regular basis, people would send me mail asking me to implement some new feature.Usually the feature was a "one-off" -- a feature that solved their particular problem. Like, "*I need to call ChangeLightBulbWindowHandleEx, but there is no ActiveX control that does so and you can't call Win32 APIs directly from script, can you add a ChangeLightBulbWindowHandleEx method to the VBScript built-in functions? It would only be like five lines of code\!"*

I'd always tell these people the same thing -- **if it is only five lines of code then go write your own ActiveX object\!** Because yes, you are absolutely right -- it would take me approximately five minutes to add that feature to the VBScript runtime library. But how many Microsoft employees does it actually take to change a lightbulb?

  - One dev to spend five minutes implementing *ChangeLightBulbWindowHandleEx.*
  - One program manager to write the specification.
  - One localization expert to review the specification for localizability issues.
  - One usability expert to review the specification for accessibility and usability issues.
  - At least one dev, tester and PM to brainstorm security vulnerabilities.
  - One PM to add the security model to the specification.
  - One tester to write the test plan.
  - One test lead to update the test schedule.
  - One tester to write the test cases and add them to the nightly automation.
  - Three or four testers to participate in an ad hoc bug bash.
  - One technical writer to write the documentation.
  - One technical reviewer to proofread the documentation.
  - One copy editor to proofread the documentation.
  - One documentation manager to integrate the new documentation into the existing body of text, update tables of contents, indexes, etc.
  - Twenty-five translators to translate the documentation and error messages into all the languages supported by Windows.The managers for the translators live in Ireland (European languages) and Japan (Asian languages), which are both severely time-shifted from Redmond, so dealing with them can be a fairly complex logistical problem.
  - A team of senior managers to coordinate all these people, write the cheques, and justify the costs to their Vice President.

None of these take very long individually, but they add up, and this is for a **simple** feature.You'll note that I haven't added all the things that Joe talks about, like what if there is a bug in those five lines of code? **That initial five minutes of dev time translates into many person-weeks of work and enormous costs,** all to save one person a few minutes of whipping up a one-off VB6 control that does what they want.Sorry, but that makes no business sense whatsoever. **At Microsoft we try very, very hard to not release half-baked software.** Getting software right -- by, among other things, ensuring that a legally blind Catalan-speaking Spaniard can easily use the feature without worrying about introducing a new security vulnerability -- is rather expensive\! But we have to get it right because when we ship a new version of the script engines, hundreds of millions of people will exercise that code, and tens of millions will program against it.

Any new feature which does not serve a large percentage of those users is essentially **stealing** valuable resources that could be spent implementing features, fixing bugs or looking for security vulnerabilities that DO impact the lives of millions of people.

UPDATE: [KC Lemson](http://blogs.technet.com/kclemson/archive/2004/05/10/129544.aspx) and [Raymond Chen](http://blogs.msdn.com/oldnewthing/archive/2004/05/13/131116.aspx) and [Chris Pratley](http://blogs.msdn.com/chris_pratley/archive/2004/01/31/65606.aspx) have opinions on this as well.

