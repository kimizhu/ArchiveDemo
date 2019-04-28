# Announcing Microsoft Roslyn June 2012 CTP

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/5/2012 12:55:00 PM

-----

Good afternoon all, I am happy to announce that we are releasing a **second** Community Technology Preview release of Roslyn, the project I actually work on, today. I am super excited\!

So, let's cut to the chase. Key facts:

  - Roslyn is a library of code analysis APIs useful for building compilers, development environments, refactoring engines and so on. It supports lexical, grammatical and semantic analysis of C\# and Visual Basic. And it is awesome.
  - This version of the CTP works well with the [Visual Studio 2012 Release Candidate that was recently made available for download](http://msdn.microsoft.com/en-us/vstudio/bb984878.aspx).
  - The C\# semantic analysis engine [now supports most, but not all, the C\# language features](http://social.msdn.microsoft.com/Forums/en-US/roslyn/thread/f5adeaf0-49d0-42dc-861b-0f6ffd731825). In particular, query expressions, anonymous types, anonymous functions and iterator blocks are now supported. The largest not-yet-implemented features are the "dynamic" feature from C\# 4 and the "await" feature from C\# 5. Nullable arithmetic *mostly* works but the code we generate is non-optimal; I haven't had time to write an optimizer yet.
  - We are giving you this sneak peek in order to get your feedback on the API design and related features such as the interactive window. **Please post any comments you have to the [Roslyn forum](http://social.msdn.microsoft.com/forums/en-us/roslyn), and not to this blog.** We have a team of awesome program managers who are gathering feedback from the forums and using it to help us tune the APIs to be as useful as possible for you all. We'll certainly take bug reports, but constructive feedback on the APIs is what we are going for here.
  - For a longer overview of this release, see [Jason's blog post](http://blogs.msdn.com/b/jasonz/archive/2012/06/05/announcing-microsoft-roslyn-june-2012-ctp.aspx). You can get all the details and download the CTP from [msdn.com/roslyn](http://msdn.com/roslyn).

