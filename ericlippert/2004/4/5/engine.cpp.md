# engine.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2004 4:00:00 PM

-----

\#include "headers.h"

ScriptEngine::ScriptEngine()  
{  
    DLLAddRef();

    this-\>m\_cref = 1;  
    this-\>m\_thread = threadNone;  
    this-\>m\_state = SCRIPTSTATE\_UNINITIALIZED;  
    this-\>m\_psite = NULL;  
    this-\>m\_pNamedItemList = NULL;  
}

ScriptEngine::~ScriptEngine()  
{  
    Assert(this-\>m\_cref == 0);

    if (this-\>m\_state \!= SCRIPTSTATE\_CLOSED && this-\>m\_state \!= SCRIPTSTATE\_UNINITIALIZED)  
    {  
        Bug("It's a very bad idea to deallocate an initialized engine.");  
    }

    this-\>Close();  
    if (NULL \!= this-\>m\_pNamedItemList)  
        delete this-\>m\_pNamedItemList;

    DLLRelease();  
}

HRESULT ScriptEngine::Create(ScriptEngine \* \* ppEngine)  
{  
    AssertOutPtr(ppEngine);

    HRESULT hr;

    \*ppEngine = NULL;  
    ScriptEngine \* pEngine = NULL;

    pEngine = new ScriptEngine();  
    if (NULL == pEngine)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    hr = NamedItemList::Create(23, \&pEngine-\>m\_pNamedItemList);  
    if (FAILED(hr))  
        goto LError;

    \*ppEngine = pEngine;  
    pEngine = NULL;

    hr = S\_OK;       

LError:

    if (NULL \!= pEngine)  
        pEngine-\>Release();  
         
    return hr;  
}

void ScriptEngine::ChangeScriptState(SCRIPTSTATE state)  
{  
    if (this-\>m\_state \!= state)  
    {  
        this-\>m\_state = state;  
        if (this-\>m\_psite \!= NULL)  
            this-\>m\_psite-\>OnStateChange(state);  
    }  
}

BOOL ScriptEngine::IsInitialized(void)  
{  
    switch(this-\>m\_state)  
    {  
    case SCRIPTSTATE\_CLOSED:  
    case SCRIPTSTATE\_UNINITIALIZED:  
        Assert(NULL == this-\>m\_psite);  
        Assert(threadNone == this-\>m\_thread);  
        return FALSE;  
    case SCRIPTSTATE\_INITIALIZED:  
    case SCRIPTSTATE\_STARTED:  
    case SCRIPTSTATE\_CONNECTED:  
    case SCRIPTSTATE\_DISCONNECTED:  
        Assert(threadNone \!= this-\>m\_thread);  
        Assert(NULL \!= this-\>m\_psite);  
        return TRUE;  
    default:  
        Bug("The script state is a bad value.");  
        return FALSE;  
    }  
}

HRESULT ScriptEngine::VerifyThread(void)  
{  
    if (this-\>IsInitialized() && this-\>m\_thread \!= GetCurrentThreadId())  
    {

        Bug("The host is in violation of the script engine threading contract. "  
            "Hosts are required to treat the engine as apartment threaded when "  
            "the engine is initialized (with some exceptions) and as rental "  
            "threaded when it is not initialized.");

        return E\_UNEXPECTED;  
    }  
    return S\_OK;  
}

HRESULT ScriptEngine::Uninitialize(void)  
{  
    HRESULT hr;  
     
    if (this-\>m\_state == SCRIPTSTATE\_CLOSED)  
    {  
        Bug("It is illegal to move a script engine from CLOSED to UNINITIALIZED");  
        // Pointless, too.  
        return E\_UNEXPECTED;  
    }

    if (this-\>m\_state == SCRIPTSTATE\_UNINITIALIZED)  
        return S\_OK;

    hr = this-\>Initialize();  
    if (FAILED(hr))  
        goto LError;

    this-\>m\_pNamedItemList-\>Reset();

    this-\>m\_psite-\>Release();  
    this-\>m\_psite = NULL;  
    this-\>m\_thread = threadNone;

    hr = S\_OK;

LError:

    return hr;  
}

HRESULT ScriptEngine::Initialize(void)  
{  
    HRESULT hr;  
     
    if (\!this-\>IsInitialized())  
    {  
        Bug("To initialize an uninitialized engine, call SetScriptSite.");  
        return E\_UNEXPECTED;  
    }

    if (this-\>m\_state == SCRIPTSTATE\_INITIALIZED)  
        return S\_OK;

    if (this-\>m\_state == SCRIPTSTATE\_CONNECTED || this-\>m\_state == SCRIPTSTATE\_DISCONNECTED)  
    {  
        hr = this-\>FullyDisconnectEventSinks();  
        if (FAILED(hr))  
            goto LError;  
    }

    hr = this-\>StopEngine();  
    if (FAILED(hr))  
        goto LError;

    hr = S\_OK;

LError:

    return hr;  
}

HRESULT ScriptEngine::Start(void)  
{  
    HRESULT hr;

    if (\!this-\>IsInitialized())  
    {  
        Bug("You cannot start an uninitialized engine.");  
        return E\_UNEXPECTED;  
    }

    if (this-\>m\_state == SCRIPTSTATE\_STARTED)  
        return S\_OK;

    if (this-\>m\_state == SCRIPTSTATE\_CONNECTED || this-\>m\_state == SCRIPTSTATE\_DISCONNECTED)  
    {  
        hr = this-\>FullyDisconnectEventSinks();  
        if (FAILED(hr))  
            goto LError;  
    }

    hr = this-\>Execute();

    if (SCRIPT\_E\_REPORTED == hr)  
        hr = S\_OK;  
    if (FAILED(hr))  
        goto LError;

    hr = S\_OK;

LError:

    return hr;  
}

HRESULT ScriptEngine::Execute(void)  
{  
    // UNDONE  
    return S\_OK;  
}

HRESULT ScriptEngine::StopEngine(void)  
{  
    // UNDONE  
    return S\_OK;  
}

HRESULT ScriptEngine::Connect(void)  
{  
    HRESULT hr;

    if (\!this-\>IsInitialized())  
    {  
        Bug("You cannot connect an uninitialized engine.");  
        return E\_UNEXPECTED;  
    }

    if (this-\>m\_state == SCRIPTSTATE\_CONNECTED)  
        return S\_OK;

    if (this-\>m\_state == SCRIPTSTATE\_DISCONNECTED)  
    {  
        hr = this-\>ReconnectEventSinks();  
        if (FAILED(hr))  
            goto LError;  
    }  
    else  
    {  
        hr = this-\>Start();  
        if (FAILED(hr))  
            goto LError;

        hr = this-\>ConnectEventSinks();  
        if (FAILED(hr))  
            goto LError;         
    }

    hr = S\_OK;

LError:

    return hr;  
}

HRESULT ScriptEngine::TemporarilyDisconnectEventSinks(void)  
{  
    if (this-\>m\_state == SCRIPTSTATE\_DISCONNECTED)  
        return S\_OK;

    if (this-\>m\_state \!= SCRIPTSTATE\_CONNECTED)  
    {  
        Bug("You cannot disconnect an engine that is not connected\!");  
        return E\_UNEXPECTED;  
    }  
     
    // UNDONE  
    return S\_OK;  
}

HRESULT ScriptEngine::FullyDisconnectEventSinks(void)  
{  
    Assert(this-\>m\_state == SCRIPTSTATE\_CONNECTED || this-\>m\_state == SCRIPTSTATE\_DISCONNECTED);

    // UNDONE  
    return S\_OK;  
}

HRESULT ScriptEngine::ConnectEventSinks(void)  
{  
    // UNDONE  
    return S\_OK;  
}

HRESULT ScriptEngine::ReconnectEventSinks(void)  
{  
    // UNDONE  
    return S\_OK;  
}

// IUnknown

STDMETHODIMP ScriptEngine::QueryInterface(REFIID riid, void \* \* ppv)  
{  
    // This method can be called on any thread at any time.  
     
    if (NULL == ppv)  
    {  
        Bug("Null out pointer");  
        return E\_POINTER;  
    }  
         
    AssertOutPtr(ppv);

    \*ppv = NULL;

    if (IsEqualIID(riid, IID\_IUnknown))  
        \*ppv = (IUnknown \*)(IActiveScript \*) this;  
    else if (IsEqualIID(riid, IID\_IActiveScript))  
        \*ppv = (IActiveScript \*) this;  
    else if (IsEqualIID(riid, IID\_IActiveScriptParse))  
        \*ppv = (IActiveScriptParse \*) this;  
    else if (IsEqualIID(riid, IID\_IActiveScriptParseProcedure2))  
        \*ppv = (IActiveScriptParseProcedure2 \*) this;  
    else if (IsEqualIID(riid, IID\_IObjectSafety))  
        \*ppv = (IObjectSafety \*) this;  
    else  
        return E\_NOINTERFACE;

    this-\>AddRef();  
    return S\_OK;  
}

STDMETHODIMP\_(ULONG) ScriptEngine::AddRef(void)  
{  
    // This method can be called on any thread at any time.  
     
    return InterlockedIncrement(\&this-\>m\_cref);  
}

STDMETHODIMP\_(ULONG) ScriptEngine::Release(void)  
{  
    // This method can be called on any thread at any time,  
    // but if this is the final release, the engine should be  
    // uninitialized.  
     
    long cref = InterlockedDecrement(\&this-\>m\_cref);  
    if (0 == cref)  
        delete this;  
    return cref;  
}

// IActiveScript

STDMETHODIMP ScriptEngine::SetScriptSite(IActiveScriptSite \* psite)  
{  
    if (NULL == psite)  
    {  
        Bug("Null site pointer");  
        return E\_POINTER;  
    }

    AssertReadPtr(psite);  
     
    if (this-\>IsInitialized())  
    {  
        Bug("It is a violation of the script engine contract to set "  
            "the site of an engine which is already initialized");  
        return E\_UNEXPECTED;  
    }  
         
    psite-\>AddRef();  
    this-\>m\_psite = psite;  
    this-\>m\_thread = GetCurrentThreadId();

    this-\>ChangeScriptState(SCRIPTSTATE\_INITIALIZED);

    return S\_OK;  
}  
     
STDMETHODIMP ScriptEngine::GetScriptSite(REFIID riid, void \* \* ppv)  
{  
    HRESULT hr;

    if (NULL == ppv)  
    {  
        Bug("Null out pointer");  
        return E\_POINTER;  
    }

    AssertOutPtr(ppv);

    \*ppv = NULL;  
     
    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        return hr;

    if (this-\>m\_psite == NULL)  
        return S\_FALSE;

    return this-\>m\_psite-\>QueryInterface(riid, ppv);  
}

STDMETHODIMP ScriptEngine::SetScriptState(SCRIPTSTATE state)  
{  
    HRESULT hr;  
     
    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        goto LError;  
     
    switch(state)  
    {  
    default:  
        Bug("The script state is a bad value.");  
        hr = E\_INVALIDARG;  
        break;  
    case SCRIPTSTATE\_CLOSED:  
        hr = this-\>Close();  
        break;  
    case SCRIPTSTATE\_UNINITIALIZED:  
        hr = this-\>Uninitialize();  
        break;  
    case SCRIPTSTATE\_INITIALIZED:  
        hr = this-\>Initialize();  
        break;  
    case SCRIPTSTATE\_STARTED:  
        hr = this-\>Start();  
        break;  
    case SCRIPTSTATE\_CONNECTED:  
        hr = this-\>Connect();  
        break;  
    case SCRIPTSTATE\_DISCONNECTED:  
        hr = this-\>TemporarilyDisconnectEventSinks();  
        break;  
    }

    if (FAILED(hr))  
        goto LError;

    this-\>ChangeScriptState(state);

    hr = S\_OK;

LError:

    return hr;  
}

STDMETHODIMP ScriptEngine::Close(void)  
{  
    HRESULT hr;

    if (this-\>m\_state == SCRIPTSTATE\_CLOSED)  
        return S\_OK;

    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        goto LError;

    hr = this-\>Uninitialize();  
    if (FAILED(hr))  
        goto LError;

    this-\>m\_pNamedItemList-\>Clear();

    this-\>ChangeScriptState(SCRIPTSTATE\_CLOSED);

    hr = S\_OK;

LError:   

    return hr;  
}

STDMETHODIMP ScriptEngine::AddNamedItem(const WCHAR \* pszName, DWORD flags)  
{

// There are six possible flags.  Suppose the named item is "foo":  
//  
// SCRIPTITEM\_ISVISIBLE     : foo is added to the namespace, so you can call  
//                            "foo.bar(123)" from the script  
// SCRIPTITEM\_ISSOURCE      : foo is an object which sources events  
// SCRIPTITEM\_GLOBALMEMBERS : calling "bar(123)" calls "foo.bar(123)".  Think  
//                            "window.alert()". These guys go into the global module.  
// SCRIPTITEM\_ISPERSISTENT  : foo survives when we go back to Uninitialized state.  
// SCRIPTITEM\_CODEONLY      : foo is just the name of a module, not an object  
// SCRIPTITEM\_NOCODE        : foo is just the name of a named item, not a code module

    HRESULT hr;

    if (NULL == pszName)  
        return E\_POINTER;

    AssertReadString(pszName);  
     
    if (\!this-\>IsInitialized())  
    {  
        Bug("It is a violation of the script engine contract to add "  
            "a named item to an uninitialized engine.");  
        return E\_UNEXPECTED;  
    }

    if ((flags & ~SCRIPTITEM\_ALL\_FLAGS) \!= 0)  
    {  
        Bug("Bad flag passed to AddNamedItem");  
        return E\_INVALIDARG;  
    }

    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        return hr;

    hr = this-\>m\_pNamedItemList-\>Add(pszName, flags);  
    if (FAILED(hr))  
        return hr;

    return S\_OK;  
}

STDMETHODIMP ScriptEngine::AddTypeLib(REFGUID rguid, DWORD major, DWORD minor,  
    DWORD flags)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::GetScriptDispatch(const WCHAR \* pszName,  
    IDispatch \* \* ppdisp)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::GetScriptState(SCRIPTSTATE \* pstate)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::GetCurrentScriptThreadID(SCRIPTTHREADID \* pthread)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::GetScriptThreadID(DWORD thread, SCRIPTTHREADID \* pthread)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::GetScriptThreadState(SCRIPTTHREADID thread,  
    SCRIPTTHREADSTATE \* pstate)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::InterruptScriptThread(SCRIPTTHREADID thread,  
    const EXCEPINFO \* pexcepinfo, DWORD flags)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::Clone(IActiveScript \* \* ppScriptEngine)  
{  
    return E\_NOTIMPL;  
}

// IActiveScriptParse

STDMETHODIMP ScriptEngine::InitNew(void)  
{  
    return S\_OK;  
}

STDMETHODIMP ScriptEngine::AddScriptlet(const WCHAR \* pszDefaultName,  
    const WCHAR \* pszCode, const WCHAR \* pszItemName, const WCHAR \* pszSubItemName,  
    const WCHAR \* pszEventName, const WCHAR \* pszDelimiter, DWORD sourceContext,  
    ULONG startingLineNumber, DWORD flags, BSTR \* pbstrName, EXCEPINFO \* pexcepinfo)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::ParseScriptText(const WCHAR \* pszCode,  
    const WCHAR \* pszItemName, IUnknown \*punkContext, const WCHAR \* pszDelimiter,  
    DWORD sourceContext, ULONG startingLineNumber, DWORD flags,  
    VARIANT \*pvarResult, EXCEPINFO \*pexcepinfo)  
{  
    return E\_NOTIMPL;  
}

// IActiveScriptParseProcedure2

STDMETHODIMP ScriptEngine::ParseProcedureText(const WCHAR \* pstrCode,  
    const WCHAR \* pstrFormalParams, const WCHAR \* pstrProcedureName,  
    const WCHAR \* pstrItemName, IUnknown \* punkContext, const WCHAR \* pstrDelimiter,  
    DWORD sourceContext, ULONG startingLineNumber, DWORD flags, IDispatch \* \* ppdisp)  
{  
    return E\_NOTIMPL;  
}

// IObjectSafety

STDMETHODIMP ScriptEngine::GetInterfaceSafetyOptions(REFIID riid,  
    DWORD \* pSupported, DWORD \* pEnabled)  
{  
    return E\_NOTIMPL;  
}

STDMETHODIMP ScriptEngine::SetInterfaceSafetyOptions(REFIID riid,  
    DWORD mask, DWORD options)  
{  
    return E\_NOTIMPL;  
}

