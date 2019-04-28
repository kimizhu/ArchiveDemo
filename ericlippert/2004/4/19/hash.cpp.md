# hash.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 6:19:00 PM

-----

\#include "headers.h"

ULONG ComputeHash(const WCHAR \* psz)  
{  
    AssertReadString(psz);  
    ULONG hash = 0;  
     
    while (\*psz \!= '  
    {  
        hash = 17 \* hash + \*psz;  
        ++psz;  
    }  
    return hash;  
}

