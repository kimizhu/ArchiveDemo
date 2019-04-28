# classfac.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2004 1:16:00 PM

-----

\#include "headers.h"

// ClassFactory

ClassFactory::ClassFactory()  
{  
    m\_cref = 1;  
}

ClassFactory::~ClassFactory()  
{  
}

HRESULT ClassFactory::Create(ClassFactory \* \* ppFactory)  
{  
    AssertOutPtr(ppFactory);  
    \*ppFactory = new ClassFactory();  
    if (NULL == \*ppFactory)  
        return E\_OUTOFMEMORY;  
    return S\_OK;  
}

// IUnknown

STDMETHODIMP ClassFactory::QueryInterface(REFIID riid, void\*\* ppv)  
{  
    if (ppv == NULL)  
        return E\_POINTER;  
         
    AssertOutPtr(ppv);

    \*ppv = NULL;

    if (IsEqualIID(riid, IID\_IUnknown))  
        \*ppv = (IUnknown \*) this;  
    else if (IsEqualIID(riid, IID\_IClassFactory))  
        \*ppv = (IClassFactory \*) this;  
    else  
        return E\_NOINTERFACE;

    this-\>AddRef();  
    return S\_OK;  
}

STDMETHODIMP\_(ULONG) ClassFactory::AddRef(void)  
{  
    return InterlockedIncrement(\&this-\>m\_cref);  
}

STDMETHODIMP\_(ULONG) ClassFactory::Release(void)  
{  
    long cref = InterlockedDecrement(\&this-\>m\_cref);  
    if (0 == cref)  
        delete this;  
    return cref;  
}

// IClassFactory

STDMETHODIMP ClassFactory::LockServer(BOOL fLock)  
{  
    if (fLock)  
        DLLAddLock();  
    else  
        DLLReleaseLock();  
    return S\_OK;  
}

STDMETHODIMP ClassFactory::CreateInstance(IUnknown \* punkOuter, REFIID riid, void \* \* ppv)  
{  
    HRESULT hr;  
     
    ScriptEngine \* pEngine = NULL;

    if (NULL == ppv)  
        return E\_POINTER;

    AssertOutPtr(ppv);  
         
    \*ppv = NULL;

    // The engine does not support aggregation  
    if (NULL \!= punkOuter)  
        return E\_INVALIDARG;

    hr = ScriptEngine::Create(\&pEngine);  
    if (FAILED(hr))  
        goto LError;  
     
    hr = pEngine-\>QueryInterface(riid, ppv);  
    if (FAILED(hr))  
        goto LError;

    hr = S\_OK;

LError:

    if (NULL \!= pEngine)  
        pEngine-\>Release();

    return hr;  
}

