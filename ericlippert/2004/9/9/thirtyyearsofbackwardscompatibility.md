# Thirty Years of Backwards Compatibility

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/9/2004 10:12:00 AM

-----

In response to my earlier series on [error handling in VBScript](http://blogs.msdn.com/ericlippert/archive/2004/08/19/217244.aspx), [Ian Griffiths blogged](http://www.interact-sw.co.uk/iangblog/2004/09/01/altaircompatible) that Reuben Harris noticed an interesting fact about VBScript error numbers -- a fact which I will shamelessly steal and blog about myself. Here's the complete VBScript **runtime** error number table.     5 Invalid procedure call or argument  
    6 Overflow  
    7 Out of memory  
    9 Subscript out of range  
   10 This array is fixed or temporarily locked  
   11 Division by zero  
   13 Type mismatch  
   14 Out of string space  
   17 Can't perform requested operation  
   28 Out of stack space  
   35 Sub or Function not defined  
   48 Error in loading DLL  
   51 Internal error  
   52 Bad file name or number  
   53 File not found  
   54 Bad file mode  
   55 File already open  
   57 Device I/O error  
   58 File already exists  
   61 Disk full  
   62 Input past end of file  
   67 Too many files  
   68 Device unavailable  
   70 Permission denied  
   71 Disk not ready  
   74 Can't rename with different drive  
   75 Path/File access error  
   76 Path not found  
   91 Object variable not set  
   92 For loop not initialized  
   94 Invalid use of Null  
  322 Can't create necessary temporary file  
  424 Object required  
  429 ActiveX component can't create object  
  430 Class doesn't support Automation  
  432 File name or class name not found during Automation operation  
  438 Object doesn't support this property or method  
  440 Automation error  
  445 Object doesn't support this action  
  446 Object doesn't support named arguments  
  447 Object doesn't support current locale setting  
  448 Named argument not found  
  449 Argument not optional  
  450 Wrong number of arguments or invalid property assignment  
  451 Object not a collection  
  453 Specified DLL function not found  
  455 Code resource lock error  
  457 This key is already associated with an element of this collection  
  458 Variable uses an Automation type not supported in VBScript  
  462 The remote server machine does not exist or is unavailable  
  481 Invalid picture  
  500 Variable is undefined  
  501 Illegal assignment  
  502 Object not safe for scripting  
  503 Object not safe for initializing  
  504 Object not safe for creating  
  505 Invalid or unqualified reference  
  506 Class not defined  
  507 An exception occurred  
32811 Element not found  
32812 The specified date is not available in the current locale's calendar There are errors missing from the table -- errors which VB6 raises that VBScript never raises, like "Return without gosub" or "Bad DLL calling convention".  And there are errors added that make no sense in VB6, like "not safe for scripting".  But more or less, it's the same as the VB6 run-time error table where possible. Obviously that's for backwards compatibility reasons. Altair BASIC was Microsoft's first product, written by Bill Gates and Paul Allen back in 1975 for the Altair 8080. (Trivia: Microsoft's corporate switchboard number 425-882-8080 is an homage to the Altair\!) Here's the Microsoft Altair BASIC error table, both parser and runtime errors:  1 NEXT without FOR  
 2 Syntax error  
 3 RETURN without GOSUB  
 4 Out of data  
 5 Illegal function call  
 6 Overflow  
 7 Out of memory  
 8 Undefined line  
 9 Subscript out of range  
10 Redimensioned array  
11 Division by zero  
12 Illegal direct  
13 Type mismatch  
14 Out of string space  
15 String too long  
16 String formula too complex  
17 Can't continue  
18 Undefined user function  
19 No RESUME  
20 RESUME without error  
21 Unprintable error  
22 Missing operand  
23 Line buffer overflow  
26 FOR without NEXT  
29 WHILE without WEND  
30 WEND without WHILE  
50 Field overflow  
51 Internal error  
52 Bad file number  
53 File not found  
54 Bad file mode  
55 File already open  
57 Disk I/O error  
58 File already exists  
61 Disk full  
62 Input past end  
63 Bad record number  
64 Bad file name  
66 Direct statement in file  
67 Too many files I had no idea that the chain of backwards compatibility went back to the original Bill & Paul code\!  30 years of backwards compatibility -- not bad\!

