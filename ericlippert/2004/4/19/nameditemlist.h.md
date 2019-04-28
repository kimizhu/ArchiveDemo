# nameditemlist.h

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 6:25:00 PM

-----

\#ifndef NAMEDITEMLIST\_H // {  
\#define NAMEDITEMLIST\_H

class NamedItemList  
{

private:

    class NamedItem  
    {  
    private:

        NamedItem();

    public:

        ~NamedItem();  
        static HRESULT Create(const WCHAR \* pszName, NamedItem \* \* ppNamedItem);  
     
        NamedItem \* m\_pNext;  
        BSTR m\_bstrName;  
        DWORD m\_flags;

        BOOL IsPersistent();  
        void Reset();  
    };

    NamedItemList();  
    NamedItem \* Find(const WCHAR \* psz);  
    Mutex \* m\_pMutex;  
    NamedItem \* \* m\_Buckets;  
    int m\_cBuckets;

public:

    ~NamedItemList();

    static HRESULT Create(int cBuckets, NamedItemList \* \* ppNamedItemList);  
    HRESULT Add(const WCHAR \* pszName, DWORD flags);  
    void Reset(void);  
    void Clear(void);  
    HRESULT Clone(NamedItemList \* \* ppNamedItemList);  
};

\#endif // NAMEDITEMLIST\_H }

