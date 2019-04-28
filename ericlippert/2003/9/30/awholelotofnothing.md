# A Whole Lot Of Nothing

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/30/2003 1:37:00 PM

-----

 

Occasionally I get questions from people who are confused over the semantics of *data that are not even there*.  Usually they've written code something like

 

 

If Blah = Nothing Then

 

 

or

 

 

If Blah = Empty Then

 

 

or

 

 

If Blah = Null Then

 

 

all three of which almost certainly do not correctly express the actual intention of the programmer.  Why does VBScript have Null, Nothing and Empty, and what are the differences between them?

 

 

Let's start with Empty.  When you declare a variable in C, the variable's value before the first assignment is undefined:

 

 

int index;

printf("%d", index);  /\* could print any integer \*/

 

 

In C, the declaration **reserves space for the variable**, but does not **clear the contents of that space**.  After all, why would it need to? You're just going to initialize it to some value yourself, right?  Why should the compiler waste time by initializing it only to have that initialization overwritten?

 

 

That might seem like a sensible attitude if you are one of those people who prefers that your program be twenty or even *thirty nanoseconds* faster in exchange for causing any accidental use of uninitialized memory to make your program's behaviour completely random. 

 

 

The designers of VB knew that their users were not hard core bit twiddling performance wonks, but rather line-of-business developers who prefer a predictable programming environment.  Thus, VB initializes variables as they are declared and eats a few processor cycles here and there.  When you declare an integer in VB, it's initialized to zero, strings are initialized to empty strings, and so on.

 

 

But what about variants?  Should an uninitialized variant be initialized to zero?  That seems bogus; why should an uninitialized variant automatically become a number? Really what we need is a special of-no-particular-type "I'm an uninitialized variant" value, and that's Empty.  And since in VBScript, all variables are variants, all variables are initialized to Empty.

 

 

What if in VB you compare an uninitialized variant to an uninitialized integer?  It seems sensible that the comparison would return True, and it does.  Empty compares as equal to 0 and the empty string, which might cause false positives in our example above.  If you need to detect whether a variable actually is an empty variant and not a string or a number, you can use IsEmpty.  (Alternatively, you could use TypeName or VarType, but I prefer IsEmpty.)

 

 

Nothing is similar to Empty but subtly different.  Empty says "I am an uninitialized variant", Nothing says "I am an object reference that refers to no object".  Since the equality operator on objects checks for equality on the default property of an object, any attempt to say If Blah = Nothing Then  is doomed to failure -- Nothing does not have a default property, so this will produce a run-time error.  To check to see if an object reference is invalid, use If Blah Is Nothing Then.

 

 

Null is weirder still.  The semantics of Null are very poorly understood, particularly amongst people who have little experience with relational databases.  Empty says "I'm an uninitialized variant", Nothing says "I'm an invalid object" and Null says "I represent a value which is not known."

 

 

Let me give an example.  Suppose you have a database of sales reports, and you ask the database "*what was the total of all sales in August*?" but one of the sales staff has not reported their sales for August yet.  What's the correct answer?  You could design the database to ignore the fact that data is missing and give the sum of the known sales, but that would be answering a different question.  The question was not "*what was the total of all known sales in August, excluding any missing data*?"  The question was "*what was the total of all sales in August*?"  The answer to that question is "**I don't know -- there is data missing**", so the database returns Null.

 

 

What happens when you say

 

 

If Blah = Null Then

 

 

?  Well, try this:

 

 

Sales = 123

WScript.Echo Sales = Null

 

 

You get not True, not False, but Null\!  Why's that?  Well, think about the semantics of it.  You're saying "is the unknown quantity equal to 123?"  The answer to that is not "yes", it's not "no", it's "I don't know what the unknown quantity is, so, uh, maybe?" 

 

 

**Nulls propagate themselves**.  Any time you numerically manipulate a Null, you get a Null right back.  Any sum containing an unknown addend has an unknown sum, obviously\!  The correct way to check for Null is much as you'd do for Empty: use IsNull (or TypeName or VarType.)

 

 

 

The sharp-eyed among you will have noticed that I never actually answered the question.  What does happens when you say  

 

If Blah = Null Then 

 

\-- does VBScript run the consequence block or the (optional) "else" block?  Obviously it has to do one of the two.  When it comes right down to it, VBScript will assume falsity in this situation.

 

 

The way JScript and JScript .NET handle nulls is a little bit weird; I'll talk about that in my next entry.

