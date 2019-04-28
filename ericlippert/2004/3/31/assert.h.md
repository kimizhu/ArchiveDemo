# assert.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2004 6:15:00 PM

-----

\#ifndef ASSERT\_H // {  
\#define ASSERT\_H  
extern BOOL Debugger();  
extern BOOL AssertProc(const char \*pszFile, long lwLine, const char \*pszMsg);  
extern BOOL IsValidReadString(const WCHAR \* psz);  
extern BOOL IsValidReadStringN(const WCHAR \* psz);  
extern BOOL IsGoodReadPtr(void \* pv, ULONG cb);  
extern BOOL IsGoodWritePtr(void \* pv, ULONG cb);

\#if DEBUG // {  
\#define AssertMsg(exp, msg) ((exp) || \!AssertProc(\_\_FILE\_\_, \_\_LINE\_\_, (msg)) || Debugger())  
\#else // } DEBUG {  
\#define AssertMsg(exp, msg)  
\#endif // DEBUG }

\#define Bug(msg)               AssertMsg(FALSE, msg)  
\#define Assert(exp)            AssertMsg(exp, \#exp)  
\#define AssertReadStringN(exp) AssertMsg(IsValidReadStringN((exp)), "bad string")  
\#define AssertReadString(exp)  AssertMsg(IsValidReadString((exp)), "bad string")  
\#define AssertOutPtr(exp)      AssertMsg(IsGoodWritePtr((exp), sizeof(void\*)), "bad out pointer")  
\#define AssertReadPtr(exp)     AssertMsg(IsGoodReadPtr((exp), sizeof(void\*)), "bad pointer")

\#endif // ASSERT\_H }

