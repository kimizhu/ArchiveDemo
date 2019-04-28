<div id="page">

# Style follows semantics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/1/2010 8:29:00 AM

-----

<div id="content">

<div class="mine">

Which is better style?

<span class="code"> </span>

bool abc;  
if (Foo())  
  abc = Bar();  
else  
  abc = false;

vs

<span class="code"> </span>

bool abc = Foo() && Bar();

?

To me, this comes down to the question “is Bar useful solely for obtaining its value, or also for its side effects?” **The stylistic choices should typically be driven by a desire to clearly communicate the semantics of the program fragment.**

The metasyntatic names are therefore making this harder to answer, not easier. Suppose the choice were in fact between:

<span class="code"> </span>

bool loginSuccessful;  
if (NetworkAvailable())  
  loginSuccessful= LogUserOn();  
else  
  loginSuccessful= false;

and

<span class="code"> </span>

bool loginSuccessful= NetworkAvailable() && LogUserOn();

I would always choose the former, because I want LogUserOn to be in a statement of its own. Statements emphasize “I am useful for my side effects”. Statements emphasize control flow and provide convenient places to put breakpoints.

If however the choice were between

<span class="code"> </span>

bool canUseCloud;  
if (NetworkAvailable())  
  canUseCloud = UserHasFreeSpaceInCloud();  
else  
  canUseCloud = false;

and

<span class="code"> </span>

bool canUseCloud = NetworkAvailable() && UserHasFreeSpaceInCloud();

I would always choose the latter; here we’re using && idiomatically to compute a value safely.  

</div>

</div>

</div>

