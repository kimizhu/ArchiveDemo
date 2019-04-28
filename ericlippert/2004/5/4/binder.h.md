# binder.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 10:01:00 AM

-----

\#ifndef BINDER\_H // {  
\#define BINDER\_H

class Binder : public IDispatch  
{

protected:

    class Name  
    {  
    public:

        Name();  
        ~Name();  
         
        HRESULT SetValue(VARIANTARG \* pvar);  
        HRESULT GetValue(VARIANT \* pvar);  
        BOOL IsFunction(void);  
        HRESULT ExecuteFunction(UINT cArgs, VARIANTARG \* rgvarArgs, VARIANT \* pvarResult);

        VARIANT m\_var;  
    };

public:

    static HRESULT Create(Binder \* \* ppBinder);

    // IUnknown  
    STDMETHOD(QueryInterface)(REFIID riid, void \* \* ppv);  
    STDMETHOD\_(ULONG,AddRef)(void);  
    STDMETHOD\_(ULONG,Release)(void);  
     
    // IDispatch  
    STDMETHOD(GetTypeInfoCount)(UINT \* pcTypeInfo);  
    STDMETHOD(GetTypeInfo)(UINT iTypeInfo, LCID lcid, ITypeInfo \* \* ppTypeInfo);  
    STDMETHOD(GetIDsOfNames)(REFIID riid, WCHAR \* \* rgpszNames, UINT cNames,  
        LCID lcid, DISPID \* rgdispids);  
    STDMETHOD(Invoke)(DISPID dispid, REFIID riid, LCID lcid, WORD flags,  
        DISPPARAMS \* pDispParams, VARIANT \* pvarResult, EXCEPINFO \* pExcepInfo,  
        UINT \* pError);

protected:

    long m\_cref;  
    DWORD m\_thread;

    Binder();  
    virtual ~Binder();

    HRESULT VerifyThread(void);  
    HRESULT GetIdOfName(const WCHAR \* pszName , DISPID \* pdispid);  
    HRESULT GetNameById(DISPID dispid, Name \* \* ppName);

};

\#endif // BINDER\_H }

