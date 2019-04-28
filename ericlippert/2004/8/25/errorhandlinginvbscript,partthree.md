# Error Handling in VBScript, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/25/2004 2:14:00 PM

-----

Apparently I've sparked a [discussion](http://lambda-the-ultimate.org/node/view/202 "http") amongst the super-geniuses of LtU on various innovative language constructs for handling errors. Fascinating stuff that I'd love to learn more about\! But I'll be less highfalutin: no doubt about it, error handling in VBScript is a pain in the rear no matter how you slice it.

This series was inspired in party by an email from a reader who was interested in the philosophy of error handling. Here's an excerpt which contrasts two approaches: (my emphasis)

I think I'm old school in saying that error handling should be very tight. Handle errors where you expect to find them. Everything else is left to fail. **I'd rather have a program end in a messy death than to blithely continue on in an unpredictable fashion**. Some of my cohorts would rather do broad error handling (whole subroutines or sections of the script). They seem to assume that only the errors they expect will happen. And even if other errors do happen, **it's better to have the script finish as best it can than to do nothing at all.**

I've talked about this [before](http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx "http") (at the bottom of the post.) As a professional developer who writes complex C\# and C++ code on a large team building a product that will be shipped to millions of people, I agree with the writer. Error handling should be built into the architecture from day one. And of course we have done so in VSTO2 -- we have developed a set of exception classes with error numbers and localizable error strings, and tried to engineer everything so that only the right errors get propagated up to the user.

But I'm working in C\# and C++, languages specifically designed for implementing complex software written by large teams. VBScript is not such a language -- it was designed for simple administration and web scripts, where often "muddle on through" is exactly what you want it to do.

For example, I have a simple script much like [this one](http://blogs.msdn.com/gstemp/archive/2004/08/10/212113.aspx "http") that I use for doing quick-and-dirty regular expression searches on my hard disk. You’d better believe that On Error Resume Next is on for that script\! If it encounters a directory or file that it cannot search because it is locked by another process, or that I don't have permission to read, or whatever, then I don't want my little twenty-line script to die horribly\! I certainly don't want to be constantly maintaining it to add new error logic as I discover more special cases. Were I writing a user-grade bulletproof hard disk searching tool that was going to be shipped in the operating system, you'd better believe I'd have error handling all over that thing, but for my scripty purposes, doing its best and muddling through is almost always plenty good enough. Use the right tool for the job.

Finally, one more implementation detail that I forgot to mention the other day. When VBScript gets certain error numbers back from calls to IDispatch objects, it sometimes takes the error number and replaces it with the equivalent VBScript error number. I have never particularly liked this feature, but VB6 does it, so we're stuck with it for backwards compatibility reasons. This can be a little confusing if you're debugging a problem -- you see one error go out of the object, but a different error is reported to the host. On the off chance that someone finds this useful, I'll put the mapping table that VBScript uses below.

Coming up next time, part two of [Riddle Me This, Google](http://blogs.msdn.com/ericlippert/archive/2004/05/11/130128.aspx), where once more I dole out advice on subjects I know little about -- love, optics and primatology -- based on questions culled from my most recent 29950 Google hits. Stay tuned\!

0x80004001 (E\_NOTIMPL)                  --\> 445 (ActionNotSupported)  
0x80004002 (E\_NOINTERFACE)              --\> 430 (OLENotSupported)  
0x80020001 (DISP\_E\_UNKNOWNINTERFACE)    --\> 438 (OLENoPropOrMethod)  
0x80020003 (DISP\_E\_MEMBERNOTFOUND)      --\> 438 (OLENoPropOrMethod)  
0x80020004 (DISP\_E\_PARAMNOTFOUND)       --\> 448 (NamedParamNotFound)  
0x80020005 (DISP\_E\_TYPEMISMATCH)        --\>  13 (TypeMismatch)  
0x80020006 (DISP\_E\_UNKNOWNNAME)         --\> 438 (OLENoPropOrMethod)  
0x80020007 (DISP\_E\_NONAMEDARGS)         --\> 446 (NamedArgsNotSupported)  
0x80020008 (DISP\_E\_BADVARTYPE)          --\> 458 (InvalidTypeLibVariable)  
0x8002000A (DISP\_E\_OVERFLOW)            --\>   6 (Overflow)  
0x8002000B (DISP\_E\_BADINDEX)            --\>   9 (OutOfBounds)  
0x8002000C (DISP\_E\_UNKNOWNLCID)         --\> 447 (LocaleSettingNotSupported)  
0x8002000D (DISP\_E\_ARRAYISLOCKED)       --\>  10 (ArrayLocked)  
0x8002000E (DISP\_E\_BADPARAMCOUNT)       --\> 450 (FuncArityMismatch)  
0x8002000F (DISP\_E\_PARAMNOTOPTIONAL)    --\> 449 (ParameterNotOptional)  
0x80020011 (DISP\_E\_NOTACOLLECTION)      --\> 451 (NotEnum)  
0x8002802F (TYPE\_E\_DLLFUNCTIONNOTFOUND) --\> 453 (InvalidDllFunctionName)  
0x80028CA0 (TYPE\_E\_TYPEMISMATCH)        --\>  13 (TypeMismatch)  
0x80028CA1 (TYPE\_E\_OUTOFBOUNDS)         --\>   9 (OutOfBounds)  
0x80028CA2 (TYPE\_E\_IOERROR)             --\>  57 (IOError)  
0x80028CA3 (TYPE\_E\_CANTCREATETMPFILE)   --\> 322 (CantCreateTmpFile)  
0x80030002 (STG\_E\_FILENOTFOUND)         --\> 432 (OLEFileNotFound)  
0x80030003 (STG\_E\_PATHNOTFOUND)         --\>  76 (PathNotFound)  
0x80030004 (STG\_E\_TOOMANYOPENFILES)     --\>  67 (TooManyFiles)  
0x80030005 (STG\_E\_ACCESSDENIED)         --\>  70 (PermissionDenied)  
0x80030008 (STG\_E\_INSUFFICIENTMEMORY)   --\>   7 (OutOfMemory)  
0x80030012 (STG\_E\_NOMOREFILES)          --\>  67 (TooManyFiles)  
0x80030013 (STG\_E\_DISKISWRITEPROTECTED) --\>  70 (PermissionDenied)  
0x8003001D (STG\_E\_WRITEFAULT)           --\>  57 (IOError)  
0x8003001E (STG\_E\_READFAULT)            --\>  57 (IOError)  
0x80030020 (STG\_E\_SHAREVIOLATION)       --\>  75 (PathFileAccess)  
0x80030021 (STG\_E\_LOCKVIOLATION)        --\>  70 (PermissionDenied)  
0x80030050 (STG\_E\_FILEALREADYEXISTS)    --\>  58 (FileAlreadyExists)  
0x80030070 (STG\_E\_MEDIUMFULL)           --\>  61 (DiskFull)  
0x800300FC (STG\_E\_INVALIDNAME)          --\> 432 (FileNotFound)  
0x80030100 (STG\_E\_INUSE)                --\>  70 (PermissionDenied)  
0x80030101 (STG\_E\_NOTCURRENT)           --\>  70 (PermissionDenied)  
0x80030103 (STG\_E\_CANTSAVE)             --\>  57 (IOError)  
0x80040154 (REGDB\_E\_CLASSNOTREG)        --\> 429 (CantCreateObject)  
0x800401E3 (MK\_E\_UNAVAILABLE)           --\> 429 (CantCreateObject)  
0x800401E6 (MK\_E\_INVALIDEXTENSION)      --\> 432 (OLEFileNotFound)  
0x800401EA (MK\_E\_CANTOPENFILE)          --\> 432 (OLEFileNotFound)  
0x800401F3 (CO\_E\_CLASSSTRING)           --\> 429 (CantCreateObject)  
0x800401F5 (CO\_E\_APPNOTFOUND)           --\> 429 (CantCreateObject)  
0x800401FE (CO\_E\_APPDIDNTREG)           --\> 429 (CantCreateObject)  
0x80070005 (E\_ACCESSDENIED)             --\>  70 (PermissionDenied)  
0x8007000E (E\_OUTOFMEMORY)              --\>   7 (OutOfMemory)  
0x80070057 (E\_INVALIDARG)               --\>   5 (IllegalFuncCall)  
0x800706BA (RPC\_S\_SERVICE\_UNAVAILABLE)  --\> 462 (ServerNotFound)  
0x80080005 (CO\_E\_SERVER\_EXEC\_FAILURE)   --\> 429 (CantCreateObject)

