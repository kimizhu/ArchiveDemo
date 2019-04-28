# Pop Quiz

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/29/2004 10:56:00 AM

-----

Every non-trivial language has "gotchas". Here's one. Consider the following JScript, which creates an object and sets some fields to zero: var x =  
{  
  0       : 0,  
  123     : 0,  
  1.23    : 0,  
  blah    : 0,  
  "dude\!" : 0,  
  "-1.23" : 0  
}; A little weird, but it works just fine. How about this? var x = { -1 : 0 }; That fails with the message "**expected identifier, string or number**". Pop quiz: what the heck is going on here? Surely that is an "identifier, string or number", no?

