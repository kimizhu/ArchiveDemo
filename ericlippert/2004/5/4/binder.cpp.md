# binder.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 10:01:00 AM

-----

\#include "headers.h"

Binder::Binder()  
{  
    DLLAddRef();  
    this-\>m\_cref = 1;  
    this-\>m\_thread = GetCurrentThreadId();  
}

Binder::~Binder(void)  
{  
    DLLRelease();  
}

HRESULT Binder::Create(Binder \* \* ppBinder)  
{  
    AssertOutPtr(ppBinder);

    \*ppBinder = new Binder();  
    if (NULL == \*ppBinder)  
        return E\_OUTOFMEMORY;

    return S\_OK;  
}

// IUnknown

STDMETHODIMP\_(ULONG) Binder::AddRef(void)  
{  
    return InterlockedIncrement(\&this-\>m\_cref);  
}

STDMETHODIMP\_(ULONG) Binder::Release(void)  
{  
    long cref = InterlockedDecrement(\&this-\>m\_cref);  
    if (0 == cref)  
        delete this;  
    return cref;  
}

STDMETHODIMP Binder::QueryInterface(REFIID riid, void \* \* ppv)  
{  
    if (NULL == ppv)  
    {  
        Bug("Null out pointer");  
        return E\_POINTER;  
    }  
         
    AssertOutPtr(ppv);

    \*ppv = NULL;

    if (IsEqualIID(riid, IID\_IUnknown))  
        \*ppv = (IUnknown \*)(IDispatch \*) this;  
    else if (IsEqualIID(riid, IID\_IDispatch))  
        \*ppv = (IDispatch \*) this;  
    else  
        return E\_NOINTERFACE;

    this-\>AddRef();  
    return S\_OK;  
}

// IDispatch

STDMETHODIMP Binder::GetTypeInfoCount(UINT \* pcTypeInfo)  
{  
    if (NULL == pcTypeInfo)  
    {  
        Bug("Null out pointer");  
        return E\_POINTER;  
    }

    \*pcTypeInfo = 1;  
    return S\_OK;  
}

STDMETHODIMP Binder::GetTypeInfo(UINT iTypeInfo, LCID lcid, ITypeInfo \* \* ppTypeInfo)  
{  
    HRESULT hr;  
//    TypeInfoBuilder \* pTypeInfoBuilder = NULL;

    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        goto LError;

    if (NULL == ppTypeInfo)  
    {  
        Bug("Null out pointer");  
        hr = E\_POINTER;  
        goto LError;  
    }

    if (0 \!= iTypeInfo)  
    {  
        Bug("We only have one type info -- why are you asking for more?");  
        hr = DISP\_E\_BADINDEX;  
        goto LError;  
    }

    hr = E\_NOTIMPL;

//  hr = TypeInfoBuilder::Create(lcid, \&pTypeInfoBuilder);  
//  if (FAILED(hr))  
//      goto LError;  
//  
//  hr = pTypeInfoBuilder-\>AddBinder(this);  
//  if (FAILED(hr))  
//      goto LError;  
//  
//  hr = pTypeInfoBuilder-\>GetTypeInfo(ppTypeInfo);  
//  if (FAILED(hr))  
//      goto LError;  
//  
//  hr = S\_OK;  
//  
LError:

//  if (NULL \!= pTypeInfoBuilder)  
//      pTypeInfoBuilder-\>Release();

    return hr;  
}

STDMETHODIMP Binder::GetIDsOfNames(REFIID riid, WCHAR \* \* rgpszNames, UINT cNames,  
    LCID lcid, DISPID \* rgdispids)  
{  
    // The first name is the name of a property or method.  The rest are names  
    // of arguments to the method.  Since we do not support invocation via named  
    // parameters, we'll decline to identify additional names.  
     
    HRESULT hr;  
    UINT iName;

    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        goto LError;

    if (IID\_NULL \!= riid)  
    {  
        return hr= DISP\_E\_UNKNOWNINTERFACE;  
        goto LError;  
    }

    if (NULL == rgdispids)  
    {  
        Bug("Bad out pointer");  
        hr = E\_POINTER;  
        goto LError;  
    }

    if (0 == cNames)  
    {  
        Bug("Why are you asking for ids of an empty list of names?");  
        hr = DISP\_E\_UNKNOWNNAME;  
        goto LError;  
    }

    for (iName = 0 ; iName \< cNames; ++iName)  
        rgdispids\[iName\] = DISPID\_UNKNOWN;

    WCHAR \* pszName = rgpszNames\[0\];  
    if (NULL == pszName)  
    {  
        Bug("Bad string argument");  
        hr = E\_POINTER;  
        goto LError;  
    }

    hr = this-\>GetIdOfName(pszName, \&rgdispids\[0\]);  
    if (FAILED(hr))  
        goto LError;

    if (cNames \> 1)  
    {  
        hr = DISP\_E\_UNKNOWNNAME;  
        goto LError;  
    }

    hr = S\_OK;

LError:

    return hr;  
}

STDMETHODIMP Binder::Invoke(DISPID dispid, REFIID riid, LCID lcid, WORD flags,  
    DISPPARAMS \* pDispParams, VARIANT \* pvarResult, EXCEPINFO \* pExcepInfo,  
    UINT \* pError)  
{  
    HRESULT hr;  
    Name \* pName;

    // The number of ways you can call Invoke wrong is enormous.  Let's  
    // check most of them up front, after we null out the return values.

    if (NULL \!= pvarResult)  
        pvarResult-\>vt = VT\_EMPTY;

    if (NULL \!= pExcepInfo)  
        memset(pExcepInfo, 0x00, sizeof EXCEPINFO);

    if (NULL \!= pError)  
        pError = 0;

    hr = this-\>VerifyThread();  
    if (FAILED(hr))  
        goto LError;

    if (IID\_NULL \!= riid)  
    {  
        hr = DISP\_E\_UNKNOWNINTERFACE;  
        goto LError;  
    }

    if (NULL == pDispParams)  
    {  
        Bug("Null dispatch parameters.");  
        hr = E\_POINTER;  
        goto LError;  
    }

    if (pDispParams-\>cArgs \< pDispParams-\>cNamedArgs)  
    {  
        Bug("More named arguments than arguments\!");  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (0 == (flags & (DISPATCH\_PROPERTYGET | DISPATCH\_PROPERTYPUT |  
        DISPATCH\_PROPERTYPUTREF | DISPATCH\_METHOD)))  
    {  
        Bug("Must pass in at least one flag.");  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (0 \!= (flags & ~(DISPATCH\_PROPERTYGET | DISPATCH\_PROPERTYPUT |  
        DISPATCH\_PROPERTYPUTREF | DISPATCH\_METHOD)))  
    {  
        Bug("Unsupported flag.");  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    BOOL fCall = (0 \!= (flags & DISPATCH\_METHOD));  
    BOOL fGet  = (0 \!= (flags & DISPATCH\_PROPERTYGET));  
    BOOL fPut  = (0 \!= (flags & (DISPATCH\_PROPERTYPUT | DISPATCH\_PROPERTYPUTREF)));

    if (fPut && fCall)  
    {  
        Bug("A property put is not a method call.");  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (fPut && fGet)  
    {  
        Bug("Cannot mix property put and property get.");  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (fPut && (0 == pDispParams-\>cNamedArgs ||  
        DISPID\_PROPERTYPUT \!= pDispParams-\>rgdispidNamedArgs\[0\]))  
    {  
        Bug("Must supply exactly one named argument when assigning a property.");  
        hr = DISP\_E\_PARAMNOTOPTIONAL;  
        goto LError;  
    }

    if (fPut && (1 \!= pDispParams-\>cNamedArgs))  
    {  
        Bug("We don't support multiple named arguments to property puts.")  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (\!fPut && (0 \!= pDispParams-\>cNamedArgs))  
    {  
        Bug("We don't support named arguments to except to property puts.")  
        hr = E\_INVALIDARG;  
        goto LError;  
    }

    if (dispid \< 0)  
    {  
        Bug("We do not support any 'special' dispatch ids.");  
        hr = DISP\_E\_MEMBERNOTFOUND;  
        goto LError;  
    }  
     
    // This code is going to get a little weird, but it does work out in the end.  
    //  
    // I hope.  
    //  
    // What we're going to do here is first take care of all the cases where  
    // we're calling the default property of an object.  Then we'll take care  
    // of all property puts, then all calls to functions.  At that point,  
    // we've exhausted everything that makes sense to have an arguments list,  
    // so the only thing left is property gets.  
    //  
    // I'll spell it out in more detail as we go.

    // You think this code is crazy, you should see the actual JScript/VBScript  
    // implementations -- which support garbage collection, property accessor  
    // functions and arrays called like methods\!

    hr = this-\>GetNameById(dispid, \&pName);  
    if (FAILED(hr))  
        goto LError;  
         
    // Case 1: We're doing some kind of function call on an object: either  
    //  
    // (a) a property get with arguments, or  
    // (b) a function call which is not a property get, or  
    // (c) a property put with arguments  
    //  
    // and the value associated with the dispid is a valid object.  
    //  
    // We simply defer to the object's implementation of Invoke and return.

    if (IsValidDispatch(\&pName-\>m\_var) &&  
        ((fGet && 0 \!= pDispParams-\>cArgs) || (fCall && \!fGet) ||  
        (fPut && 1 \!= pDispParams-\>cArgs)))  
    {  
        hr = InvokeDispatch(pName-\>m\_var.pdispVal, DISPID\_VALUE, IID\_NULL, lcid,  
            flags, pDispParams, pvarResult, pExcepInfo, pError);  
        goto LError;  
    }

    // Case 2:  
    // (a) We're doing a property put with arguments, but since case 1 didn't  
    //     handle it, this must not be a dispatch method.  Error out.  
    // (b) We're doing a property put without arguments.  Assign the property

    if (fPut)  
    {  
        if (pDispParams-\>cArgs \> 1)  
        {  
            hr = DISP\_E\_TYPEMISMATCH;  
            goto LError;  
        }

        hr = pName-\>SetValue(\&pDispParams-\>rgvarg\[0\]);  
        goto LError;  
    }

    // Case 3:  We're doing a function call on a function.

    if (pName-\>IsFunction())  
    {  
        if (\!fCall)  
        {  
            hr = DISP\_E\_TYPEMISMATCH;  
            goto LError;  
        }

        hr = pName-\>ExecuteFunction(pDispParams-\>cArgs, pDispParams-\>rgvarg, pvarResult);  
        goto LError;  
    }

    // Case 4: We've got arguments, but this isn't a dispatch object or  
    // a function.  What the heck?

    if (0 \!= pDispParams-\>cArgs)  
    {  
        hr = DISP\_E\_TYPEMISMATCH;  
        goto LError;  
    }

    // Case 5: We're doing a property get with no arguments.  Just return  
    // the value.  (We can't be calling a function here.)  Failing to  
    // provide room for the return value is dumb, but not illegal.

    if (fGet)  
    {  
        if (NULL == pvarResult)  
            hr = S\_OK;  
        else  
            hr = pName-\>GetValue(pvarResult);  
        goto LError;  
    }

    // Case 6: Something has gone terribly wrong.

    hr = E\_INVALIDARG;

LError:

    return hr;  
}

HRESULT Binder::VerifyThread(void)  
{  
    if (this-\>m\_thread \!= GetCurrentThreadId())  
    {  
        Bug("The host is in violation of the script engine threading contract. "  
            "A script dispatch object is STA threaded.");  
        return E\_UNEXPECTED;  
    }  
    return S\_OK;  
}

HRESULT Binder::GetIdOfName(const WCHAR \* pszName, DISPID \* pdispid)  
{  
    AssertOutPtr(pdispid);

    return E\_NOTIMPL;  
}

HRESULT Binder::GetNameById(DISPID dispid, Name \* \* ppName)  
{  
    AssertOutPtr(ppName);

    return E\_NOTIMPL;  
}

Binder::Name::Name()  
{  
    this-\>m\_var.vt = VT\_EMPTY;  
}

Binder::Name::~Name()  
{  
    VariantClear(\&this-\>m\_var);  
}

HRESULT Binder::Name::SetValue(VARIANTARG \* pvar)  
{  
    AssertReadPtr(pvar);  
    return VariantCopyInd(\&this-\>m\_var, pvar);  
}

HRESULT Binder::Name::GetValue(VARIANT \* pvar)  
{  
    AssertOutPtr(pvar);  
    return VariantCopy(pvar, \&this-\>m\_var);  
}

BOOL Binder::Name::IsFunction(void)  
{  
    // UNDONE  
    return FALSE;  
}

HRESULT Binder::Name::ExecuteFunction(UINT cArgs, VARIANTARG \* rgvarArgs, VARIANT \* pvarResult)  
{  
    Assert(this-\>IsFunction());  
    return E\_NOTIMPL;  
}

