# Roslyn September 2012 CTP is now available

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2012 11:58:15 AM

-----

I am super excited to announce that we have just released a third "Community Technology Preview" of Roslyn. Roslyn, in case you have not heard, is the code name for the project I work on; we are re-architecting the C\# and VB compilers so that they are no longer "black boxes" where code goes in, a miracle happens, and then IL comes out. Rather, the box is now glass and you can use the lexical, syntactic and semantic analysis engines that we write for your own purposes.

We have implemented semantic analysis of most of the C\# and VB language features now. On the C\# side we are through most of the C\# 3 features; we still lack "dynamic" from C\# 4 and "await" from the recently-released C\# 5. We've also made many changes (hopefully all improvements) to the APIs. For a complete list of the updates, to access the Roslyn question-and-answer forum, or to download it and try it for yourself, go to [msdn.com/roslyn](http://msdn.com/roslyn).

We would love to get your feedback on the [forum](http://social.msdn.microsoft.com/forums/en-us/roslyn) or on [connect.microsoft.com/visualstudio](http://blogs.msdn.comconnect.microsoft.com/visualstudio) about what you do and do not like about the APIs; that's why we do these technology previews so often. (Please leave feedback on the forum rather than as a comment to this blog; we have a team of program managers who read the forums.) I hope you enjoy this latest CTP; we enjoyed building it.

