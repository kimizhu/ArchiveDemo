<div id="page">

# Hostile Exception Filter vs Benign Assert: Who will win?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/8/2004 2:30:00 PM

-----

<div id="content">

I got a number of good responses to last week's challenge, including some that pointed out potential flaws in my solution.  See the comments for details. The question I posed is as follows: does asserting a permission before calling a method which could throw an exception allow an exception filter up the stack to obtain the effect of the assert?  Let's reason it through.  Suppose the permission asserted is FooPermission.  Assembly Bravo.DLL is the hostile, partially-trusted assembly which is NOT granted FooPermission.  Your code is assembly Charlie.DLL, which is fully trusted.  Bravo calls Charlie.  Charlie asserts FooPermission, throws an exception, and Bravo's hostile exception filter runs.  It then calls code which demands FooPermission.  Bravo has not been granted FooPermission, so the stack walk terminates immediately, never getting to the assert.  No hole here. Are we done with this analysis?  No, there are still at least two more ways that Bravo could be devious. First scenario: Again, partially trusted hostile assembly Bravo.DLL is NOT granted FooPermission.  It calls benign, partially trusted assembly Alpha.DLL which **has** been granted FooPermission.  Alpha calls Charlie, which again, asserts the permission and throws the exception.  Stupidly enough, benign Alpha has been written to have an exception filter which calls code that demands FooPermission. Now the stack walk terminates before it gets to Bravo, when normally it would fail.  Bravo has lured Alpha and Charlie into working together to do something Bravo is not allowed to do. Second scenario: Switch it up; in this scenario, partially trusted benign assembly Alpha.DLL is NOT granted FooPermission.  Partially trusted hostile assembly Bravo.DLL IS granted FooPermission but is n**ot granted the right to assert** FooPermission.  Alpha calls Bravo.  Bravo calls Charlie.  Charlie asserts FooPermission and throws an exception.  Bravo's exception filter calls code which demands FooPermission.  Normally Bravo would be denied because Alpha is on the stack, but Bravo has lured Charlie into asserting the permission, terminating the stack walk before Alpha.  Bravo has lured Charlie into preventing Alpha from being checked. Both of these scenarios are unlikely but possible and therefore **the CLR suppresses the assert while the exception filter runs**.  It does not suppress deny or permit-only stack annotations.  **You do not have to worry about exception filters being used to take advantage of unreverted assertions.**

</div>

</div>

