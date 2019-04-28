# engine.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2004 4:01:00 PM

-----

\#ifndef ENGINE\_H // {  
\#define ENGINE\_H

const DWORD threadNone = 0xFFFFFFFF;

// The error has been reported to the host via IActiveScriptSite::OnScriptError  
const HRESULT SCRIPT\_E\_REPORTED = 0x80020101L;

class ScriptEngine :  
    public IActiveScript,  
    public IActiveScriptParse,  
    public IActiveScriptParseProcedure2,  
    public IObjectSafety  
{  
private:

    long m\_cref;  
    IActiveScriptSite \* m\_psite;  
    SCRIPTSTATE m\_state;  
    DWORD m\_thread;

    NamedItemList \* m\_pNamedItemList;

    ScriptEngine();  
    ~ScriptEngine();

    void ChangeScriptState(SCRIPTSTATE state);

    BOOL IsInitialized(void);  
    HRESULT VerifyThread(void);  
    HRESULT Uninitialize(void);  
    HRESULT Initialize(void);  
    HRESULT Start(void);  
    HRESULT Execute(void);  
    HRESULT StopEngine(void);  
    HRESULT Connect(void);  
    HRESULT TemporarilyDisconnectEventSinks(void);  
    HRESULT FullyDisconnectEventSinks(void);  
    HRESULT ConnectEventSinks(void);  
    HRESULT ReconnectEventSinks(void);  
     
public:

    static HRESULT Create(ScriptEngine \* \* ppEngine);

    // IUnknown  
    STDMETHOD(QueryInterface)(REFIID riid, void \* \* ppv);  
    STDMETHOD\_(ULONG,AddRef)(void);  
    STDMETHOD\_(ULONG,Release)(void);

    // IActiveScript  
    STDMETHOD(SetScriptSite)(IActiveScriptSite \* psite);  
    STDMETHOD(GetScriptSite)(REFIID riid, void \* \* ppsite);  
    STDMETHOD(SetScriptState)(SCRIPTSTATE state);  
    STDMETHOD(GetScriptState)(SCRIPTSTATE \* pstate);  
    STDMETHOD(Close)(void);  
    STDMETHOD(AddNamedItem)(const WCHAR \* pszName, DWORD flags);  
    STDMETHOD(AddTypeLib)(REFGUID rguid, DWORD major, DWORD minor, DWORD flags);  
    STDMETHOD(GetScriptDispatch)(const WCHAR \* pszName, IDispatch \* \* ppdisp);  
    STDMETHOD(GetCurrentScriptThreadID)(SCRIPTTHREADID \* pthread);  
    STDMETHOD(GetScriptThreadID)(DWORD thread, SCRIPTTHREADID \* pthread);  
    STDMETHOD(GetScriptThreadState)(SCRIPTTHREADID thread, SCRIPTTHREADSTATE \* pstate);  
    STDMETHOD(InterruptScriptThread)(SCRIPTTHREADID thread,  
        const EXCEPINFO \* pexcepinfo, DWORD flags);  
    STDMETHOD(Clone)(IActiveScript \* \* ppScriptEngine);

    // IActiveScriptParse  
    STDMETHOD(InitNew)(void);  
    STDMETHOD(AddScriptlet)(const WCHAR \* pszDefaultName,  
        const WCHAR \* pszCode, const WCHAR \* pszItemName,  
        const WCHAR \* pszSubItemName, const WCHAR \* pszEventName,  
        const WCHAR \* pszDelimiter, DWORD sourceContext,  
        ULONG startingLineNumber, DWORD flags, BSTR \* pbstrName,  
        EXCEPINFO \* pexcepinfo );  
    STDMETHOD(ParseScriptText)(const WCHAR \* pszCode, const WCHAR \* pszItemName,  
        IUnknown \* punkContext, const WCHAR \* pszDelimiter, DWORD sourceContext,  
        ULONG     startingLineNumber, DWORD flags, VARIANT \* pvarResult,  
        EXCEPINFO \* pexcepinfo);

    // IActiveScriptParseProcedure2  
    STDMETHOD(ParseProcedureText)(const WCHAR \* pszCode,  
        const WCHAR \* pszFormalParams, const WCHAR \* pszProcedureName,  
        const WCHAR \* pszItemName, IUnknown \* punkContext,  
        const WCHAR \* pszDelimiter, DWORD sourceContext,  
        ULONG startingLineNumber, DWORD flags, IDispatch \* \* ppdisp);  
         
    // IObjectSafety  
    STDMETHOD(GetInterfaceSafetyOptions)(REFIID riid,  
        DWORD \* pSupportedOptions, DWORD \* pEnabledOptions);  
    STDMETHOD(SetInterfaceSafetyOptions)(REFIID riid, DWORD mask, DWORD options);  
};

\#endif // ENGINE\_H }

