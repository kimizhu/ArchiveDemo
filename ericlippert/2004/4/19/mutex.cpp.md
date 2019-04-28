# mutex.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 6:22:00 PM

-----

\#include "headers.h"

Mutex::Mutex()  
{  
    m\_fInitialized = FALSE;  
}

HRESULT Mutex::Create(Mutex \* \* ppMutex)  
{  
    AssertOutPtr(ppMutex);

    HRESULT hr;  
    BOOL fSuccess;  
    DWORD error;  
    Mutex \* pMutex = NULL;

    pMutex = new Mutex();  
    if (NULL == pMutex)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    fSuccess = InitializeCriticalSectionAndSpinCount(\&pMutex-\>m\_criticalsection, 0);  
    if (\!fSuccess)  
    {  
        error = GetLastError();  
        hr = HRESULT\_FROM\_WIN32(error);  
        goto LError;  
    }

    pMutex-\>m\_fInitialized = true;  
    \*ppMutex = pMutex;  
    pMutex = NULL;  
    hr = S\_OK;

LError:

    if (NULL \!= pMutex)  
        delete pMutex;

    return hr;  
}

Mutex::~Mutex()  
{  
    if (this-\>m\_fInitialized)  
        DeleteCriticalSection(\&this-\>m\_criticalsection);  
}

void Mutex::Enter(void)  
{  
    EnterCriticalSection(\&this-\>m\_criticalsection);  
}

void Mutex::Leave(void)  
{  
    LeaveCriticalSection(\&this-\>m\_criticalsection);  
}

