# Hard Core Denotational Semantics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 10:55:00 PM

-----

Some of the readers of the Lambda blog ([http://lambda.weblogs.com/discuss/msgReader$8778?mode=topic\&y=2003\&m=9\&d=18](http://lambda.weblogs.com/discuss/msgReader%248778?mode=topic&y=2003&m=9&d=18) 

) were discussing my earlier throwaway line about Waldemar Horwat.   

 

 

*The august Waldemar Horwat -- who was at one time the lead Javascript developer at AOL-Time-Warner-Netscape -- once told me that he considered Javascript to be just another syntax for Common Lisp. I'm pretty sure he was being serious.* 

 

 

One user commented: 

 

 

Mozilla's CVS tree still contains the original implementation of Javascript... written in Common Lisp.

 

 

Now, I can't look at the mozilla sources for legal reasons, so I can't say for sure.  However, if you look at the drafts of the ECMAScript 4 specification that Waldemar was writing when he worked at Netscape, you'll see that he uses this denotational semantics metalanguage to describe the operation of the ECMAScript language.  (This stands in marked contrast to the vague operational semantics used for the same purpose in the ECMAScript 1, 2 and 3 specifications.)

 

 

I vaguely recall that Waldemar had built a reference implementation of ECMAScript 4 in his metalanguage, and an implementation of the metalanguage in Common Lisp.  (Like I said, that guy is hard core.)  I hypothesize that this is the thing that the Lambda reader was talking about.  If someone could confirm or deny my hypothesis for me, I'd be interested to know.  It is unfortunate that I'm unable to look at this stuff, as I'm sure it would be fascinating.

