# invoke.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 10:04:00 AM

-----

\#include "headers.h"

HRESULT InvokeDispatch(IDispatch \* pdisp, DISPID dispid, REFIID riid,  
    LCID lcid, WORD flags, DISPPARAMS \* pDispParams, VARIANT \* pvarResult,  
    EXCEPINFO \* pExcepInfo, UINT \* pError)  
{  
    AssertReadPtr(pdisp);

    HRESULT hr;

    // We must addref the pointer before the invocation.  Why?  Consider  
    // this scenario:  the invocation calls a method which calls back  
    // into the script engine, which triggers a garbage collection, which  
    // does the final release on the dispatch object.  Then control returns  
    // back to the call to Invoke, which dereferences its "this" pointer  
    // and promptly crashes.

    pdisp-\>AddRef();

    hr = pdisp-\>Invoke(DISPID\_VALUE, riid, lcid, flags, pDispParams,  
        pvarResult, pExcepInfo, pError);

    pdisp-\>Release();

    return hr;  
}

BOOL IsValidDispatch(VARIANT \* pvar)  
{  
    AssertReadPtr(pvar);  
    return VT\_DISPATCH == pvar-\>vt && NULL \!= pvar-\>pdispVal;  
}

