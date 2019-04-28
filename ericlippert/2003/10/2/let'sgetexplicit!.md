# Let's Get Explicit\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/2/2003 1:57:00 PM

-----

A reader asked me yesterday if there was a way to detect "at compile time" (ie, before the code runs) whether a JScript program contained misspelled variables.  I'm sure we've all experienced the pain of 

 

 

var inveigledFroboznicator = new Froboznicator();

// ...

print(inviegledFroboznicator.frabness);

 

 

The sooner bugs can be caught, the better, obviously.  We catch bugs like missing braces and unterminated strings before the script even runs, so why can't we catch use of undeclared identifiers?  Doesn't VBScript do that with Option Explicit?

 

 

Actually, no, it doesn't.  **Visual Basic** detects undeclared identifiers at compile time, but **VBScript** does not catch them until runtime.  The reason is because of the way the browser name lookup rules work.  Specifically:

 

 

\* The window object is an **expando** object.  You can add new properties to it at runtime.

\* All globally scoped variables and methods in a script block are automatically aggregated onto the window expando.

 

 

(Aside: a little known fact is that VBScript allows you to get around the second rule, though why you'd want to is beyond me. It is legal to put the Private keyword on a global declaration; such declarations will not be subsumed onto the window object.)

 

 

So far there's nothing stopping us from finding undeclared identifiers at compile time, but then we add:

 

 

\* The window object's top-level members are implicitly visible.

 

 

And now our dream of compile time analysis vanishes.  ANY identifier can be added to window at any time by another script block and therefore any identifier is potentially valid in every script block.  Add in the fact that the expando can be expanded with a string, and you end up with situations in which no amount of compile time analysis can possibly determine whether an identifier is legal or not.  Here's a really silly example.

 

 

\<html\>

  \<script language="jscript"\>

    function dothething()

    {

      window\[NewVar\] = 123

    }

  \</script\>

  \<script language="vbscript"\>

    Option Explicit

    Dim NewVar

    NewVar = InputBox("Type in a word") 

    ' type in "blah"

    dothething()

    window.alert blah

  \</script\>

\</html\>

 

 

This creates a new property on the window object which is not known until the user decides what to type.  Since the property is accessible without the window. prefix, there's no way to know whether the blah identifier is legal *until the program actually runs*. 

 

Notice that JScript uses the variable declared in VBScript -- it is part of the window object because it is a global, in keeping with our rules laid out above.

 

Visual Basic does not allow access to top-level members of an object without qualification, so it knows that when it sees an undeclared variable, it really is a mistake.  (Which reminds me -- I wanted to tell you guys about ways to misuse the with block, but that's another post.)

 

 

I suppose that we could have added a feature to JScript like VBScript's Option Explicit, which catches this problem at run time, but we didn't.  As I mentioned on Sept 22nd, JScript already throws a runtime error when fetching an undeclared variable.  There is still a potential bug due to misspelling on a variable set, so be careful out there.

 

Also, as Peter reminded me, the JScript .NET compiler in compatibility mode can be used to detect things like undeclared variables, provided that you're willing to have cases like the one above show up as false positives.

 

 

One more thing before I actually go do some real work: an interesting semantic difference between Visual Basic and VBScript is caused by the fact that the declaration check is put off until run time.  **Because an undeclared variable causes a run time error**, On Error Resume Next **hides undeclared variables\!** So again, be careful out there\!  Don't rely on Option Explicit to save you next time you type with mittens on, because if you suppress errors, guess what?  Errors are suppressed\!

