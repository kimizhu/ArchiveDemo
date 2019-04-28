# assert.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2004 6:16:00 PM

-----

\#include "headers.h"

static BOOL IsGoodPtr(void \* pv, ULONG cb, DWORD dwFlags)  
{  
    DWORD dwSize;  
    MEMORY\_BASIC\_INFORMATION meminfo;

    if (NULL == pv)  
        return FALSE;

    memset(\&meminfo, 0x00, sizeof meminfo);  
    dwSize = VirtualQuery(pv, \&meminfo, sizeof meminfo);  
    // If pv is a kernel-mode address then this may return zero for security reasons.  
    // In that event it is certainly NOT a valid read pointer.

    if (0 == dwSize)  
        return FALSE;

    if (MEM\_COMMIT \!= meminfo.State)  
        return FALSE;

    if (0 == (meminfo.Protect & dwFlags))  
        return FALSE;

    if (cb \> meminfo.RegionSize)  
        return FALSE;

    if ((unsigned)((char \*)pv - (char \*)meminfo.BaseAddress) \> (unsigned)(meminfo.RegionSize - cb))  
        return FALSE;

    return TRUE;  
}

BOOL IsGoodReadPtr(void \* pv, ULONG cb)  
{  
    return IsGoodPtr(pv, cb, PAGE\_READONLY | PAGE\_READWRITE | PAGE\_WRITECOPY |  
        PAGE\_EXECUTE\_READ | PAGE\_EXECUTE\_READWRITE | PAGE\_EXECUTE\_WRITECOPY);  
}

BOOL IsGoodWritePtr(void \* pv, ULONG cb)  
{  
    return IsGoodPtr(pv, cb, PAGE\_READWRITE | PAGE\_WRITECOPY |  
        PAGE\_EXECUTE\_READWRITE | PAGE\_EXECUTE\_WRITECOPY);  
}

BOOL Debugger()  
{  
\#if DEBUG // {  
    DebugBreak();  
\#endif // DEBUG }  
    return TRUE;  
}

BOOL AssertProc(const char \* pszFile, LONG lwLine, const char \* pszMsg)  
{  
    LONG sid;  
    const int cch = 512;  
    char pch\[cch\];  
    \_snprintf(pch, cch - 1, "Assert (%s line %ld): %s", pszFile, lwLine, pszMsg);  
    pch\[cch - 1\] = '  
    sid = MessageBoxA(NULL, pch,  
        "Assert\! (Y = Ignore, N = Debugger, C = Quit)",  
        MB\_SYSTEMMODAL | MB\_YESNOCANCEL | MB\_ICONHAND);  
    switch (sid)  
    {  
    default:  
        // Ignore  
        break;  
    case IDNO:  
        // Debug  
        return TRUE;  
    case IDCANCEL:  
        // Quit  
        FatalAppExitA(0, "Fatal Error Termination");  
        break;  
    }  
    return FALSE;  
}

BOOL IsValidReadString(const WCHAR \* psz)  
{  
    if (NULL == psz)  
        return FALSE;  
    return IsGoodReadPtr((void\*)psz, 2);  
}

BOOL IsValidReadStringN(const WCHAR \* psz)  
{  
    if (NULL == psz)  
        return TRUE;  
    return IsValidReadString(psz);  
}

