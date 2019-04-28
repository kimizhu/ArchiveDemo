# Checking For Script Syntax Errors

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/11/2005 1:35:00 PM

-----

A reader asked me recently whether there was a way to check a chunk of JScript or VBScript for syntax errors without actually running the code. I'm sure that there are many third-party tools which you could find that do this. If you have your own script host, you can do it yourself quite easily. The trick is to initialize the engine by setting the script site, but do not do anything that would move the engine from initialized into started state. (Note that attempting to evaluate an expression moves the engine to started state.) If the engine is initialzed but not started then calling ParseScriptText will check the text passed in for syntax errors but will not run the script. Rather, if the script compiles successfully, it is simply marked as "needs to run when the engine is started".  Unfortunately the script engines do not have an error-recovering parser, so they will detect only the first syntax error and then bail out. Still, better than nothing.

