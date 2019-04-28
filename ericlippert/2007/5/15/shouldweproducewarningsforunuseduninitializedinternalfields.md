# Should we produce warnings for unused/uninitialized internal fields?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/15/2007 1:34:00 PM

-----

You may have noticed as you've been developing C\# applications that if you have an internal field of an internal type which you never use, we give you a warning about it. In fact, there are a number of warnings that we give for such fields, depending upon whether the field was never read or written, initialized but not read, or read but not initialized (and therefore always set to its default value.)

Now, obviously we ought not to give similar warnings for public fields on public types. The thing doing the reading or writing is highly likely to be an external assembly which may not even have been written yet, so it would be silly to give a warning. (And similarly, we suppress this warning on internals if we are building a module that is going to be linked to another module; the reads and writes might be in the other module.  We also suppress it if the internal members of the assembly are visible to an external assembly.)

The C\# 2.0 compiler was a bit weak though. If you had an unused public field on an internal class, the compiler would say, well, that's a public field, suppress the warning. But a "public" field on an internal class is not actually visible to anything in the outside world (again, modulo the parenthetical above) so really we could produce the warnings here.

There were a bunch of cases like that. A few months ago I tightened up the C\# 3.0 compiler so that we now detect and warn for all the possible cases where we can actually deduce that the field is not used/not initialized.

Naturally this means that people started getting warnings on code which had previously compiled without warnings, so other teams at Microsoft that test pre-release builds of the compilers started to notice. This change has raised a few questions amongst users on other teams, which might motivate changing this behaviour again. It is particularly hard on users who compile with "all warnings are errors" turned on.

Two scenarios in particular have been brought to my attention.

First, more and more classes are now being written specifically to be used with private reflection and/or lightweight codegen. That is, something outside of the C\# language is going to be manipulating the fields of a particular class, and it is by design to have the field unread/unwritten by the static portion of the program. It's irksome to have a warning about something which is by design.

Second, more and more classes are now being written where a field is initialized but never read, but the field is initialized to a finalizable object, and the finalizer effectively “reads” the field and does work on it.

There are a number of possible responses to these scenarios:

1\) Roll back the behaviour to that of C\# 2.0. That doesn’t actually solve any of these problems; we still potentially give warnings which the user doesn’t want.  But we give warnings in fewer scenarios.  We can also make the plausible statement “this is what the compiler has always done, you can live with it”. 

The downside of course is that we would be eliminating warnings which the user may want. The whole point of this exercise in the first place was to produce useful warnings.

2\) Do nothing. Tell the people who are raising these objections to live with it.  They can turn off the warning programmatically if they want. We strike a blow for language purity and ignore the real-world objections of people who are using features external to the language like reflection.

3\) Give the "unread" warning only if the field is of a type known to have no finalizer. (Or, weaker, suppress the "unread" warning only if the field is of a type known to have a finalizer.)

4\) Eliminate all the warnings altogether.  Declare a new design principle: warnings should be only for patterns which are highly likely to be  dangerous mistakes.  The warnings in question are just helpful reminders that you might have dead code to eliminate.  FXCOP produces this warning already, so if people want to see it, they can use FXCOP or the static analysis tool of their choice.

5\) Create some new attributes that let people document how a field is intended to be used. Be smart about looking at the attribute, and if we see attributes which say "this struct is intended for use in p/invoke", or "this field is to be accessed via private reflection", and so on, then suppress the warnings.

6\) Do something else that Eric hasn't thought of.

Anyone have thoughts or opinions on this?

Thanks\!

