# Why are unused using directives not a warning?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/25/2010 7:00:00 AM

-----

As I’ve discussed before, we try to reserve warnings for only those situations where we can say with almost certainty that the code is broken, misleading or useless. One reason for trying to ensure that warnings are not “false positives” is that we don’t ever want to encourage someone to take working, correct code and break it so as to remove the warning. Another is that since many people compile with “warnings are errors” turned on, we do not want to introduce a whole lot of unnecessary build breakages should the warning turn out to be unwarranted.

An unused using directive is almost certainly not “broken”, and it’s probably not misleading. It certainly is useless though. Why not produce a warning for this, so that you can remove the useless directive from your program?

Suppose we make an unused using into a warnings. Suppose you do have “warnings are errors” turned on. And suppose you create a new console project in Visual Studio.  We generate you this code:

 

using System;  
using System.Text;  
using System.Linq; class Program  
{  
  static void Main(string\[\] arguments)  
  {  
  }  
}

… which doesn’t compile. Which is better, generating code that doesn't compile, or omitting the using directives and making you type "using System;" the first time you try to use Console.WriteLine?

Either way is a really bad user experience. So, no such feature.

