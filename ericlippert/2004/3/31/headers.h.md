# headers.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2004 6:13:00 PM

-----

\#ifndef HEADERS\_H // {  
\#define HEADERS\_H

// InitializeCriticalSectionAndSpinCount requires WinNT 4.0 SP3 or better.  
// UNDONE: Add code to registration, etc, that verifies that we are on the  
// UNDONE: right OS.

\#define \_WIN32\_WINNT 0x0403

\#include \<windows.h\>  
\#include \<objidl.h\>  
\#include \<comcat.h\>  
\#include \<objsafe.h\>  
\#include \<activscp.h\>  
\#include \<stdio.h\>  
\#include "guids.h"  
\#include "dllmain.h"  
\#include "classfac.h"  
\#include "mutex.h"  
\#include "hash.h"  
\#include "nameditemlist.h"  
\#include "engine.h"  
\#include "assert.h"  
\#include "binder.h"  
\#include "invoke.h"

\#endif // HEADERS\_H }

