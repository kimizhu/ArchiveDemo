# FYI, C\# 2.0 Has A Breaking Change in Enum Subtraction

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/19/2005 5:19:00 PM

-----

A customer brought to my attention the other day that the C\# 2.0 beta release has a breaking change from the previous release. Namely, this code

enum E : byte {  
  A = 1,  
  B = 2  
}; // . . . E a = E.A;  
E b = E.B;  
int j = a - b; sets j to -1 in the previous release but to 255 in the upcoming release. First off, let me say that we regret the breaking change. We agonize over all breaking changes because we know the pain that they cause customers. We also regret introducing the bug in the first place, thereby forcing us to choose between continuing to be in violation of the specification and breaking existing code. Sorry about all that. Second, I should describe why exactly the original behaviour is in violation of the C\# specification. It's pretty straightforward. Start with section 7.7.5: Every enumeration type implicitly provides the following predefined operator, where E is the enum type, and U is the underlying type of E: U operator –(E x, E y); This operator is evaluated exactly as (U)((U)x – (U)y) That clearly means that the assignment above should have the same semantics as int j = (byte)((byte)a-(byte)b)); C\# defines only four built-in subtraction operators: int operator –(int x, int y);  
uint operator –(uint x, uint y);  
long operator –(long x, long y);  
ulong operator –(ulong x, ulong y); There is an implicit conversion from byte to all four types, so we must select the best one.  According to section 7.4.2.3 the int version is the best one (because signed is preferable to unsigned and int goes to long but long does not go to int.)  So what we generate here is the equivalent of: int j = (byte)((int)(byte)a-(int)(byte)b)); The conversions from E to byte to int will go off without a hitch, and the subtraction will result in an int set to -1.  That then gets cast to byte. What happens when we try to cast a computed-at-runtime integer to a byte? Section 7.5.12 says For non-constant expressions (expressions that are evaluated at run-time) that are not enclosed by any checked or unchecked operators or statements, the default overflow checking context is unchecked unless external factors (such as compiler switches and execution environment configuration) call for checked evaluation. Therefore this is an unchecked cast, and -1 goes to 255 as a byte. That then gets converted back to an int during the assignment. Third, I should talk a bit about the process we go through when making breaking changes like this. The change was made to the C\# 2.0 compiler on the 14<sup>th</sup> of January 2004, six months before beta 1, and one of the reasons we try to push betas out really early is to get feedback on whether breaking changes like this affect millions, thousands, or dozens of people. Since to my knowledge the first customer to run into a break contacted us this week, and we're only taking "the product electrocutes millions of users" bug fixes right now, unfortunately this one does not make the bar for choosing backwards compatibility over correctness. I feel bad about that, but I hope you understand our reasoning here. We've got to ship this thing\! We'll also make sure that a Knowledge Base article describing the problem gets written.

