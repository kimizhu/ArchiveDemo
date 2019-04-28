# invoke.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 10:04:00 AM

-----

\#ifndef INVOKE\_H // {  
\#define INVOKE\_H

extern HRESULT InvokeDispatch(IDispatch \* pdisp, DISPID dispid, REFIID riid,  
    LCID lcid, WORD flags, DISPPARAMS \* pDispParams, VARIANT \* pvarResult,  
    EXCEPINFO \* pExcepInfo, UINT \* pError);

extern BOOL IsValidDispatch(VARIANT \* pvar);

\#endif // INVOKE\_H }

