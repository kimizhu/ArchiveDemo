<div id="page">

# Error Handling In VBScript, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/23/2004 10:41:00 AM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The way VBScript implements the error semantics I mentioned [last time](http://blogs.msdn.com/ericlippert/archive/2004/08/19/217244.aspx) is sort of interesting.  Today I'll talk a bit about the implementation details, and then next time I'll finish up by talking about some philosophy of error handling.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Before I go on, if you haven't already, read my recent post [Spot The Defect](http://blogs.msdn.com/ericlippert/archive/2004/08/17/215935.aspx "http") to see how </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> returns error information in an </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">EXCEPINFO</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">OK, so we know that when an error occurs we are going to have at the very least an </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">HRESULT</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, and possibly some additional information -- strings describing the error, and so on.  When VBScript's </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> caller gets an error it records a copy of the error in an internal buffer and returns a [special error code](http://blogs.msdn.com/ericlippert/archive/2003/10/22/53267.aspx "http"), </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">SCRIPT\_E\_RECORDED</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  That way the script engine knows to not attempt to record the error again, but rather to look at the recorded buffer state when it needs information about the real error.  Script engine users should never see this error number.  You'll only see it if you're watching EAX in the debugger during a script error.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

**<span style="FONT-WEIGHT: bold; FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Note that both </span><span style="FONT-WEIGHT: bold; FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">On Error</span><span style="FONT-WEIGHT: bold; FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> statements clear the recorded error information buffer.</span>**

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Once the error winds its way back to the interpreter, the interpreter does a [long jump](http://blogs.msdn.com/ericlippert/archive/2003/10/16/53232.aspx "http") to an error handling routine which saves off additional information that the interpreter knows.  For instance, what is the name of the last-accessed variable -- because some error messages include information like that.  The interpreter then checks to see if we're in 'resume next' mode, in which case it cleans up the stack, moves the instruction pointer to the next beginning-of-statement marker, and restarts the interpreter.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">There is some additional goo that happens in here to make script debugging scenarios work.  I won't go into that -- a discussion of all the ways that script errors and the debugger interact would be lengthy indeed.  There is also code to handle weird scenarios -- like, a JScript block with a try-catch calls a VBScript block which calls a JScript block, which throws an exception -- but I won't go into those either.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">If the engine is not in 'resume next' mode then it reports the error to the host, and returns another special code, </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">SCRIPT\_E\_REPORTED</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.  This error indicates to the host that an error occurred but that the host already knows about it.  This is intended to forestall this problem:  suppose you have a JScript block that calls a VBScript block which causes an unhandled error.  **<span style="FONT-WEIGHT: bold">Both engines are going to see an error code</span>**.  The JScript engine does not know that it called the VBScript engine, and hence that the host already knows about the error.  The JScript engine just knows that it made a call and an error code came back.  The VBScript engine needs some way to tell the JScript engine to treat this like an error, but also make sure that JScript does not tell the host to pop up another dialog box saying that there was a script error.  Without such a mechanism, you can construct scenarios in which the same error is reported to the user an arbitrarily large number of times.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The implementation of the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">Err</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> object should now be transparent -- it simply looks at the information which the engine recorded at the time of the error.  The error number, description, and so on, are reported as they were reported to us by the object which failed.  If there was no such information reported, well, you're out of luck then.</span>

</div>

</div>

