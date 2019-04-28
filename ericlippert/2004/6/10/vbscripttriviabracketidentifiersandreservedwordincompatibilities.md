# VBScript Trivia: Bracket Identifiers and Reserved Word Incompatibilities

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/10/2004 11:20:00 AM

-----

I want to spend some time addressing the security implications of private reflection in response to recent comments, but that is a big topic that will take some time, and work is absolutely crazy busy right now.  Design Change Request Season opens today -- our last chance to get approved design changes (as opposed to mere bug fixes) into Whidbey before the betas go out.  And as I'm on holiday at the end of the month, there's a lot to do in little time.   Therefore, I might not get to it soon.  Today, I've got something to get off my chest. 

 I got a question yesterday from someone who was trying to parse a config file.  He was confused by a typo.  The code was supposed to say 

 If LineText = "\[Global\]" Then 

 The typo was that he forgot the quotes 

 If LineText = \[Global\] Then 

 OK, that's easy mistake to make when you're typing fast, you get a parse error, go to that line, put the quotes in, right?  Except here's the confusing part: **this code compiled and ran. ** Incorrectly, obviously, but it ran.  What the heck? 

 Indeed, that's perfectly legal. In VBScript, here are the rules for what makes a legal identifier: 

 IDENTIFIER        : BRACKETIDENTIFIER or NORMALIDENTIFIER  
NORMALIDENTIFIER  : LETTER followed by 0 to 254 IDCHARs, but not a RESERVEDWORD  
BRACKETIDENTIFIER : \[ 0 to 255 BRACKETCHARs \]  
LETTER            : a-z, A-Z  
IDCHAR            : a-z, A-Z, 0-9, \_  
BRACKETCHAR       : any Unicode character except newline, linefeed, the null string terminator and  \]  
RESERVEDWORD      : And As Boolean ByRef Byte ByVal Call Case Class Const Currency Debug Dim Do Double Each Else ElseIf Empty End EndIf Enum Eqv Event Exit False For Function Get Goto If Imp Implements In Integer Is Let Like Long Loop LSet Me Mod New Next Not Nothing Null On Option Optional Or ParamArray Preserve Private Public RaiseEvent ReDim Rem Resume RSet Select Set Shared Single Static Stop Sub Then To True Type TypeOf Until Variant WEnd While With Xor, reserved words are case-insensitive 

 That bracket syntax is kind of weird, isn't it?  One reason to enable this syntax is so that you can write programs that look like this: 

 \[Hey, how're you doing dude? Working hard, or hardly working?\] = \[Ha, ha, ha, that's very witty Eric\!\] 

 There are also two *good* reasons.  

 First, it lets you name a method after a reserved word. 

 Function Previous()  
End Function 

 Function Next()  
End Function 

 Uh oh, that fails to compile.  But aha, 

 Function \[Next\]() 

 is legal.  This also lets you call methods on third-party objects which might be reserved words.  Imagine the irksomeness if, say, IE had a method raiseEvent which you could then not call from VBScript.  You never know what strange thing someone is going to put into an object model. 

 The other reason this is handy is because it enables developers who want to use characters which are not common in English in their identifiers.  Accented characters, Chinese characters, etc. 

 The rules in JScript are slightly different, but I'll go into them another time. 

 Which brings me to what I wanted to get off my chest.  While I'm on the subject of reserved words, this seems like a good time to publicly document something that has been irking me for a long time.  We tried to make VBScript a subset of VB6, but unfortunately there are a few "gotchas" -- accidental differences which cause a particular VBScript program to not be a legal VB6 program.  One of those gotchas is the fact that the reserved word sets are different.  You'll notice above that VBScript reserves RaiseEvent, even though there is no RaiseEvent feature in VBScript.  That's because (a) we anticipated that we might have to add it some day, and (b) "Dim RaiseEvent" isn't legal VB6, so it shouldn't be legal VBScript either. 

 There is only one word that goes "the other way" -- an identifier that is illegal in VBScript but legal in VB6 -- and that's "Class".  I felt bad about doing that, but without reserving Class it is impossible to determine whether Class Foo means "call a method with one argument" or "introduce a new class" without creating enormous bogosity in the parser. 

 For the record, the words that are legal identifiers in VBScript but illegal in VB6 are: 

 Abs AddressOf Any Array Attribute CBool CByte CCur CDate CDec CDbl CDecl CInt Circle CLng Close CSng CStr CVar CVErr Date Decimal Declare DefBool DefByte DefCur DefDate DefDec DefDblDefInt DefLng DefObj DefSng DefStr DefVar DoEvents Erase FixFriend Global GoSub Input InputB Int LBound Len LenB LINEINPUT Local Lock Open Print PSet Put Return Scale Seek Sgn Spc String Tab UBound Unlock VB\_Base VB\_Control VB\_Creatable VB\_Customizable VB\_Description VB\_Exposed VB\_Ext\_KEY VB\_HelpID VB\_Invoke\_Func VB\_Invoke\_Property VB\_Invoke\_PropertyPut VB\_Invoke\_PropertyPutRef VB\_MemberFlags VB\_Name VB\_PredeclaredId VB\_ProcData VB\_TemplateDerived VB\_VarDescription VB\_VarHelpID VB\_VarMemberFlags VB\_VarProcData VB\_UserMemId VB\_VarUserMemId VB\_GlobalNameSpace WithEvents Write 

 Once we failed to reserve them in VBScript 1.0, we couldn't reserve them at any subsequent time without risking breaking existing scripts, which we did not want to do unless there was a clear benefit (such as adding classes).  Argh. 

There are other VB6 incompatibilities, some of them truly obscure, that I'll document at a later date I'm sure.

