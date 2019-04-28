# nameditemlist.cpp

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2004 6:24:00 PM

-----

\#include "headers.h"

NamedItemList::NamedItemList()  
{  
    this-\>m\_cBuckets = 0;  
    this-\>m\_Buckets = NULL;  
    this-\>m\_pMutex = NULL;  
}

NamedItemList::~NamedItemList()  
{  
    this-\>Clear();  
    if (NULL \!= this-\>m\_Buckets)  
        delete\[\] this-\>m\_Buckets;  
    if (NULL \!= this-\>m\_pMutex)  
        delete this-\>m\_pMutex;  
}

HRESULT NamedItemList::Create(int cBuckets, NamedItemList \* \* ppNamedItemList)  
{  
    AssertOutPtr(ppNamedItemList);  
    Assert(cBuckets \> 0);

    HRESULT hr;  
    int iBucket;  
    NamedItemList \* pNamedItemList = NULL;

    \*ppNamedItemList = NULL;

    pNamedItemList = new NamedItemList();  
    if (NULL == pNamedItemList)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    hr = Mutex::Create(\&pNamedItemList-\>m\_pMutex);  
    if (FAILED(hr))  
        goto LError;

    pNamedItemList-\>m\_Buckets = new NamedItem\*\[cBuckets\];  
    if (NULL == pNamedItemList-\>m\_Buckets)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    pNamedItemList-\>m\_cBuckets = cBuckets;

    for (iBucket = 0 ; iBucket \< cBuckets ; ++iBucket)  
        pNamedItemList-\>m\_Buckets\[iBucket\] = NULL;

    \*ppNamedItemList = pNamedItemList;  
    pNamedItemList = NULL;

    hr = S\_OK;

LError:

    if (NULL \!= pNamedItemList)  
        delete pNamedItemList;

    return hr;  
}

void NamedItemList::Clear(void)  
{  
    NamedItem \* pNamedItem;  
    NamedItem \* pNext;  
    int iBucket;

    // This could get called due to out-of-memory failure at creation of the mutex.  
    if (NULL \!= this-\>m\_pMutex)  
        this-\>m\_pMutex-\>Enter();  
         
    if (this-\>m\_Buckets \!= NULL)  
    {  
        for (iBucket = 0 ; iBucket \< this-\>m\_cBuckets ; ++iBucket)  
        {  
            pNamedItem = m\_Buckets\[iBucket\];  
            while (pNamedItem \!= NULL)  
            {  
                pNext = pNamedItem-\>m\_pNext;  
                delete pNamedItem;  
                pNamedItem = pNext;  
            }  
            m\_Buckets\[iBucket\] = NULL;  
        }  
    }

    if (NULL \!= this-\>m\_pMutex)  
        this-\>m\_pMutex-\>Leave();  
}

HRESULT NamedItemList::Add(const WCHAR \* pszName, DWORD flags)  
{  
    HRESULT hr;

    int iBucket;  
    NamedItem \* pNamedItemOld;  
    NamedItem \* pNamedItem = NULL;

    this-\>m\_pMutex-\>Enter();

    pNamedItemOld = this-\>Find(pszName);  
    if (NULL \!= pNamedItemOld)  
    {  
        if (pNamedItemOld-\>m\_flags == flags)  
        {  
            Bug("You've added the same named item twice.  That's legal, but weird. "  
                "You might have a bug.");  
            hr = S\_OK;  
        }  
        else  
        {  
            Bug("It's a violation of the script engine contract to add the same "  
                "named item twice with different flags.");  
            hr = E\_UNEXPECTED;  
        }  
        goto LError;  
    }

    hr = NamedItem::Create(pszName, \&pNamedItem);  
    if (FAILED(hr))  
        goto LError;

    pNamedItem-\>m\_flags = flags;  
    iBucket = ComputeHash(pNamedItem-\>m\_bstrName) % this-\>m\_cBuckets;  
    pNamedItem-\>m\_pNext = this-\>m\_Buckets\[iBucket\];  
    this-\>m\_Buckets\[iBucket\] = pNamedItem;  
    pNamedItem = NULL;  
     
    hr = S\_OK;

LError:

    if (NULL \!= pNamedItem)  
        delete pNamedItem;

    this-\>m\_pMutex-\>Leave();

    return hr;  
}

NamedItemList::NamedItem \* NamedItemList::Find(const WCHAR \* psz)  
{  
    AssertReadString(psz);  
     
    int iBucket;  
    NamedItem \* pNamedItem;  
     
    iBucket = ComputeHash(psz) % this-\>m\_cBuckets;

    for(pNamedItem = this-\>m\_Buckets\[iBucket\] ; NULL \!= pNamedItem; pNamedItem = pNamedItem-\>m\_pNext)  
    {  
        if (0 == wcscmp(psz, pNamedItem-\>m\_bstrName))  
            break;  
    }

    return pNamedItem;  
}

void NamedItemList::Reset(void)  
{  
    int iBucket;  
    NamedItem \* pNamedItem;  
    NamedItem \* pNext;  
    NamedItem \* \* ppPrevNext;

    this-\>m\_pMutex-\>Enter();

    for (iBucket = 0 ; iBucket \< this-\>m\_cBuckets ; ++iBucket)  
    {  
        ppPrevNext = \&this-\>m\_Buckets\[iBucket\];  
        pNamedItem = this-\>m\_Buckets\[iBucket\];  
        while (NULL \!= pNamedItem)  
        {  
            pNext = pNamedItem-\>m\_pNext;  
            if (pNamedItem-\>IsPersistent())  
            {  
                pNamedItem-\>Reset();  
                ppPrevNext = \&pNamedItem-\>m\_pNext;  
            }  
            else  
            {  
                \*ppPrevNext = pNext;  
                delete pNamedItem;  
            }  
            pNamedItem = pNext;  
        }  
    }  
     
    this-\>m\_pMutex-\>Leave();  
}

HRESULT NamedItemList::Clone(NamedItemList \* \* ppNamedItemList)  
{  
    AssertOutPtr(ppNamedItemList);  
     
    HRESULT hr;  
    int iBucket;  
    NamedItemList \* pNamedItemList = NULL;  
    NamedItem \* pNamedItem;

    \*ppNamedItemList = NULL;

    this-\>m\_pMutex-\>Enter();

    hr = NamedItemList::Create(this-\>m\_cBuckets, \&pNamedItemList);  
    if (FAILED(hr))  
        goto LError;

    for (iBucket = 0 ; iBucket \< this-\>m\_cBuckets ; ++iBucket)  
    {  
        for (pNamedItem = this-\>m\_Buckets\[iBucket\] ; NULL \!= pNamedItem ;  
            pNamedItem = pNamedItem-\>m\_pNext)  
        {  
            if (\!pNamedItem-\>IsPersistent())  
                continue;  
            hr = pNamedItemList-\>Add(pNamedItem-\>m\_bstrName, pNamedItem-\>m\_flags);  
            if (FAILED(hr))  
                goto LError;  
        }  
    }

    \*ppNamedItemList = pNamedItemList;  
    pNamedItemList = NULL;

    hr = S\_OK;  
     
LError:

    if (NULL \!= pNamedItemList)  
        delete pNamedItemList;

    this-\>m\_pMutex-\>Leave();

    return hr;  
}

NamedItemList::NamedItem::NamedItem()  
{  
    this-\>m\_pNext = NULL;  
    this-\>m\_bstrName = NULL;  
    this-\>m\_flags = 0;  
}

NamedItemList::NamedItem::~NamedItem()  
{  
    SysFreeString(this-\>m\_bstrName);  
}

HRESULT NamedItemList::NamedItem::Create(const WCHAR \* pszName, NamedItem \* \* ppNamedItem)  
{  
    AssertReadString(pszName);  
    AssertOutPtr(ppNamedItem);

    HRESULT hr;

    NamedItem \* pNamedItem = NULL;  
    \*ppNamedItem = NULL;

    pNamedItem = new NamedItem();  
    if (NULL == pNamedItem)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    pNamedItem-\>m\_bstrName = SysAllocString(pszName);  
    if (NULL == pNamedItem-\>m\_bstrName)  
    {  
        hr = E\_OUTOFMEMORY;  
        goto LError;  
    }

    \*ppNamedItem = pNamedItem;  
    pNamedItem = NULL;  
    hr = S\_OK;  
     
LError:

    if (NULL \!= pNamedItem)  
        delete pNamedItem;

    return hr;  
}

BOOL NamedItemList::NamedItem::IsPersistent()  
{  
    return (this-\>m\_flags & SCRIPTITEM\_ISPERSISTENT) == 0 ? FALSE : TRUE;  
}

void NamedItemList::NamedItem::Reset(void)  
{  
}

