# dllmain.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2004 1:17:00 PM

-----

\#ifndef DLLMAIN\_H // {  
\#define DLLMAIN\_H

extern void DLLAddRef(void);  
extern void DLLRelease(void);  
extern void DLLAddLock(void);  
extern void DLLReleaseLock(void);

\#endif // DLLMAIN\_H }

