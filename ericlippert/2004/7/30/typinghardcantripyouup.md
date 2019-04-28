# Typing Hard Can Trip You Up

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/30/2004 11:31:00 AM

-----

Speaking of weird JScript gotchas, here's a weird VBScript gotcha that I alluded to [earlier](http://blogs.msdn.com/ericlippert/archive/2004/07/15/184431.aspx "http://blogs.msdn.com/ericlippert/archive/2004/07/15/184431.aspx").   foo = "1"  
bar = 1  
If "1" = 1   Then Print "True\!" Else Print "False\!"  
If "1" = bar Then Print "True\!" Else Print "False\!"  
If foo = 1   Then Print "True\!" Else Print "False\!"  
If foo = bar Then Print "True\!" Else Print "False\!" 

 This prints out "True\!" for the first three and "False\!" for the last. 

 My psychic powers tell me that you are now thinking "What the heck is up with that?"  

 This weirdness is for compatibility with some similar weirdness in VB6.  VB6 violates an important principle in programming language design, namely that **the semantics of an operation should be the same whether the type information about the operands was known at compile time or run time**.  VB6 has different semantics for operations involving variants than for operations involving "hard typed" values when comparing strings to numbers. (Booleans count as numbers -- as I discussed earlier, they are treated as -1 and 0.)  

 But that should have no impact on VBScript, right?  Because in VBScript, *everything* is a variant, so there should be no such issue.  Well, not exactly.  All variables are treated as though they were declared as variant, sure.  And internally, all data is stored in variants.  But as far as comparison operations go, VBScript follows VB6's lead and treats **literals** as hard-typed values. 

 The relevant comparison rules in VB6/VBScript go like this: 

  - Hard string ~ hard number: convert string to number, compare numbers
  - Hard string ~ soft number: convert number to string, compare strings
  - Soft string ~ hard number: convert string to number, compare numbers
  - Soft string ~ soft number: any string is greater than any number

 Though they violate the principle that semantics should be consistent regardless of when the type information is deduced, the middle two rules do make *some* sense **-- the "soft" side is converted into the type of the "hard" side for the comparison**.  The first and last rules are both arbitrary, and either is defensible.  What I don't understand is **why the first and last aren't consistent with each other**.  That just seems egregiously wrong to me.  Someone who knows more about the history of VB than I do will have to chime in here\!  This has always irked me about VB6/VBScript, but there's nothing I can do about it now.  (And of course these rules also violate the **transitive property** of the equality operator, which is irksome in itself.) 

 It gets even weirder when you consider Boolean/string comparisons: 

 bobble = "True"  
robble = True  
If "True" = True   Then Print "True\!" Else Print "False\!"  
If "True" = robble Then Print "True\!" Else Print "False\!"  
If bobble = True   Then Print "True\!" Else Print "False\!"  
If bobble = robble Then Print "True\!" Else Print "False\!" 

 That produces "True\!" for the first two, "False\!" for the last two.  Again, what the hey?   Why is this different from the case above? 

 That's a bug.  VB6 does what you'd expect, and is consistent with the integer behaviour.  We forgot to emit code that marks hard-typed Booleans as hard-typed.  Therefore VBScript uses the rules for soft typing regardless of whether the Boolean is a literal or not.  This is yet another on the long list of small, unintended deviations from the VB6 subset. 

Unfortunately, by the time we discovered the bug it was too late -- the bug had shipped to customers.  We considered fixing it, but realized that it was more important to not take the chance of breaking existing scripts than to fix this rather unimportant incompatibility with VB6.  (That wasn't the only time we passed on fixing a bug because the fix would break backwards compatibility.  But that's another story.)

